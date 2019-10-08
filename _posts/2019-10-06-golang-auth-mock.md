---
layout: post
title: "Mocking Golang with Interfaces In Real Life"
date: 2019-10-06 10:00:00
description: "Learn how to do mock testing with Golang and Interfaces with a real life example using one of Go's Spotify clients."
image: "/img/test_tube.png"
---

Authentication code usually results in an IO call, like reading a file or making a network request. These operations are villains in the _unit test_ world 'cause they create external _dependencies_. Imagine you are at the airport unable to run your tests without an Internet connection. Or if your CI system stops working due to lack of permissions to write a specific file to a specific place, halting the whole pipeline.

Writing reliable unit tests means avoiding IO operations like the plague.

![Tweet taking about unit test](/img/unit_test_tweet.png)

BUT we can't prevent _our code_ to use external resources. Cat pics need to stored somewhere. How should unit tests run without spreading its tentacles around the network?

One answer is _mocking_. Mocks are objects created to mimic the behavior of real objects in a controlled way. As an example, if the `Authenticate(user User) bool` function executes a remote call to fulfill its purpose, it can be substituted by a custom _mock_ during the test with controlled logic and output. To test the system reaction in a failed login, it just returns `false`.

In a statically typed language like Golang, is there a way to change functions on the fly? 🤔

The composite answer is **separation of concerns** and **dependency injection** – mainly using `interface` – where IO bound code is isolated as much as possible and injected into functions or methods that depend on it.

Come on, let me show you in real life.

## Mocking [zmb3/spotify](https://github.com/zmb3/spotify) Authentication
Last week I was working on a side project using Spotify's API and the [zmb3/spotify](https://github.com/zmb3/spotify) client. Its authentication method presented a perfect playground to demonstrate mocking.

Let's say we want to get a playlist's name from an ID. Here's the simplest way to do it:
```go
// main.go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	"github.com/zmb3/spotify"
	"golang.org/x/oauth2/clientcredentials"
)

func main() {
	config := &clientcredentials.Config{
		ClientID:     os.Getenv("SPOTIFY_ID"),
		ClientSecret: os.Getenv("SPOTIFY_SECRET"),
		TokenURL:     spotify.TokenURL,
	}
	token, err := config.Token(context.Background())
	if err != nil {
		log.Fatalf("couldn't get token: %v", err)
	}

	client := spotify.Authenticator{}.NewClient(token)

	const PLAYLIST_ID spotify.ID = "4OyKDT6cLw96G7bd8nTfxD"
	results, err := client.GetPlaylist(PLAYLIST_ID)
	if err != nil {
		log.Fatalf("couldn't get playlist: %v", err)
	}

	fmt.Println(results.Name)
}
```

So far, nothing is being mocked. To run the code above you will need a pair of [Spotify's](https://developer.spotify.com/) API key and secret, loaded in the environment variables:
```bash
$ export SPOTIFY_ID=<your id>
$ export SPOTIFY_SECRET=<your secret>
```

Running it:
```bash
$ go run main.go
Brunchies
```

🎉!

By the way, [Brunchies](https://open.spotify.com/playlist/4OyKDT6cLw96G7bd8nTfxD) is a real playlist. The very best one.

## Separation of Concerns
The code is quite convoluted. A single `main()` function is doing all the work. Unit testing becomes hard. Hell, impossible. A call to `main()` in the test suit will do multiple network requests to Spotify's API right away.

Let's break the code apart:

```go
// main.go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	"github.com/zmb3/spotify"
	"golang.org/x/oauth2/clientcredentials"
)

func main() {
	const PLAYLIST_ID spotify.ID = "4OyKDT6cLw96G7bd8nTfxD"

	client := newSpotifyClient()
	name := getPlaylistName(client, PLAYLIST_ID)

	fmt.Println(name)
}

func newSpotifyClient() *spotify.Client {
	config := &clientcredentials.Config{
		ClientID:     os.Getenv("SPOTIFY_ID"),
		ClientSecret: os.Getenv("SPOTIFY_SECRET"),
		TokenURL:     spotify.TokenURL,
	}
	token, err := config.Token(context.Background())
	if err != nil {
		log.Fatalf("couldn't get token: %v", err)
	}

	client := spotify.Authenticator{}.NewClient(token)

	return &client
}

func getPlaylistName(client *spotify.Client, playlistID spotify.ID) string {
	result, err := client.GetPlaylist(playlistID)
	if err != nil {
		log.Fatalf("couldn't get playlist: %v", err)
	}

	return result.Name
}
```

Nice! Two distinct methods were defined:
- `newSpotifyClient()` authenticates and returns a client to be used in subsequent operations, like getting playlist information
- `getPlaylistName()` well, gets a playlist name 🤷‍♀️

Note: the later accepts a `*spotify.Client` to do the Spotify request, this is the beginning of the dependency injection pattern. The client is being _injected_ into the function that will use it. Could we inject a mock one? 🤔

Before answering the question, take a look at this simple test:
```go
// main_test.go
package main

import (
	"testing"

	"github.com/zmb3/spotify"
)

func Test_NewGetPlaylistName(t *testing.T) {
	os.Setenv("SPOTIFY_ID", "fake id")
	os.Setenv("SPOTIFY_SECRET", "fake secret")

	client := newSpotifyClient()

	name := getPlaylistName(client, "whatever")

	if name != "whatever" {
		t.Errorf("expected %s, got %s", "whatever", name)
	}
}
```

Run it:
```bash
$ go test
2019/10/07 12:36:23 couldn't get token: oauth2: cannot fetch token: 400 Bad Request
Response: {"error":"invalid_client","error_description":"Invalid client"}
exit status 1
```

The program returns an error, trying to authenticate with our fake `SPOTIFY_ID` and `SPOTIFY_SECRET`. We don't want that.

## Interfaces to the rescue!
Imagine we could create a mock for `client spotify.Client` with the same `GetPlaylist()` method and signature and _inject_ it into in the `getPlaylistName()` method during the test.

Since Golang is statically typed – and with the current implementation – only `*spotify.Client` can be passed into `getPlaylistName()`. It needs to be refactored to accept _any type_ with a `GetPlaylist()` method – like a mock client.

Fortunately, Golang has the concept of `interface`: it is a type with a set of methods signatures. If _any_ type implements those methods, it _satisfies_ the interface and can be recognized by the interface's _type_. This is a very simple explanation, you can get some more depth [here](https://www.golang-book.com/books/intro/9).

The code below shows the interface implementation:
```go
// main.go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	"github.com/zmb3/spotify"
	"golang.org/x/oauth2/clientcredentials"
)


// 1
type spotifyClient interface {
	GetPlaylist(playlistID spotify.ID) (*spotify.FullPlaylist, error)
}

func main() {
	const PLAYLIST_ID spotify.ID = "4OyKDT6cLw96G7bd8nTfxD"

	client := newSpotifyClient()
	name := getPlaylistName(client, PLAYLIST_ID)

	fmt.Println(name)
}

func newSpotifyClient() *spotify.Client {
	config := &clientcredentials.Config{
		ClientID:     os.Getenv("SPOTIFY_ID"),
		ClientSecret: os.Getenv("SPOTIFY_SECRET"),
		TokenURL:     spotify.TokenURL,
	}
	token, err := config.Token(context.Background())
	if err != nil {
		log.Fatalf("couldn't get token: %v", err)
	}

	client := spotify.Authenticator{}.NewClient(token)

	return &client
}

// 2
func getPlaylistName(client spotifyClient, playlistID spotify.ID) string {
	result, err := client.GetPlaylist(playlistID)
	if err != nil {
		log.Fatalf("couldn't get playlist: %v", err)
	}

	return result.Name
}
```

- *1*: Creates the interface with the same `GetPlaylist()` signature from `spotify.Client`
- *2*: Accepts the `client` parameter using the `spotifyClient` interface type

Cool! I promise that everything still works 😅, just run `go run main.go`. 

Back to the tests. Now, we can create a `mockSpotifyClient` struct implementing the `GetPlaylist()` method with a controlled output. Since it satisfies the `spotifyClient` interface, it can be used in the `getPlaylistName()` function.

```go
// main_test.go
package main

import (
	"testing"

	"github.com/zmb3/spotify"
)

// 1
type mockSpotifyClient struct{}

// 2
func (m *mockSpotifyClient) GetPlaylist(playlistID spotify.ID) (*spotify.FullPlaylist, error) {
	// 3
	return &spotify.FullPlaylist{
		SimplePlaylist: spotify.SimplePlaylist{
			Name: "whatever",
		},
	}, nil
}

func Test_NewGetPlaylistName(t *testing.T) {
    // 4
    client := &mockSpotifyClient{}
    
    // 5
	name := getPlaylistName(client, "whatever")

	if name != "whatever" {
		t.Errorf("expected %s, got %s", "whatever", name)
	}
}
```

Step by step:
- *1*: Creates an empty `struct` 
- *2*: Satisfies its interface by attaching a `GetPlaylist()` method to it
- *3*: Returns a controlled input, already expected by our test
- *4*: Instantiates the `mockSpotifyClient`, no need for fake API keys anymore
- *5*: Since `client` implements the `spotifyClient` interface, it can be _injected_. When executed in `main.go`, it will return our fake playlist ✨

The proof:
```bash
$ go test
PASS
```

Isn't it awesome? 🎉

---

Here are some tips to write code like this:
- If you are still experimenting – or have a sandbox API – write the tests actually using the API to see the basics working
- Once you've identified how where the IO operations happen, isolate them, like in `getSpotifyClient()`
- After understanding which methods your "client" uses, abstract them in an interface
- If you have multiple "clients" – for Spotify, AWS, etc – create an `Env` holding all the interfaces, something like the following:

```go
// env.go
type Env struct {
    spotifyClient spotifyClientInterface
    awsClient     awsClientInterface
}
func (e *Env) Load() {
    // initialize all the clients
}

// aws.go
func awsCall(client awsClientInterface) {}

// main.go
e := Env{}
e.Load()

awsCall(e.awsClient)
```

Then, in the unit tests, just mock whatever `awsClientInterface` expects and voilà.
