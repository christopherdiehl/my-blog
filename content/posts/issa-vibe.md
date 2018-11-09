---
title: "Issa Vibe"
date: 2018-11-09T09:57:27-05:00
draft: false
keywords:
- golang
- spotify
- webapp
---


# Building a music suggesting webapp in barebones Go

For some additional background, the team I work with has a weekly *Talking Code* where one member talks about something they've worked on and would like to share with the group. Given my frequent use of Golang outside of work, I decided to make a web application that would randomly suggest a song given a genre. I also wanted to use as few libraries as possible to hide any package magic. For those who are interested in the code right away, feel free to check it out [here](https://github.com/christopherdiehl/issa-vibe)


## Finished Product
---
Before user selects the genre: 
![Before user selects genre](/issa-vibe-0.png)

After user selects the Hip Hop genre and clicks submit: 
![After user selects genre](/issa-vibe-1.png)

## How it was built
---

The webapp runs on a plain http server and relies on the `html/template` library to pass in the genre and to provide the front end the randomly selected song id. The song ids were taken from Spotify using their API located [here](https://developer.spotify.com/documentation/web-api/) and were stored in a map[genre][[]songs]. This allows the application to be even simpler because it doesn't have to make any API calls, it only needs to select one of hte stored song ids and pass it to the html page which relies on the spotify widget accepting a song id for display.

The code to generate the random song id and the link needed for the spotify widget are below:
```
var url strings.Builder
genreTracks := genreTrackMap[genre]
songIndex := rand.Intn(len(genreTracks))
url.WriteString("https://open.spotify.com/embed/track/")
url.WriteString(genreTracks[songIndex])
```


## Finished

I hope you enjoyed the post and please feel free to checkout the entire codebase [here](https://github.com/christopherdiehl/issa-vibe)