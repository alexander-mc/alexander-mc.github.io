---
layout: post
title:      "How to access Spotify’s Search API with React"
date:       2021-03-12 17:37:39 -0500
permalink:  how_to_access_spotify_s_search_api_with_react
---


### Context
***

This post is for developers seeking to integrate Spotify’s Search API into a React-based app using Spotify’s Client Credentials Flow, one of four [Spotify authorization flows](https://developer.spotify.com/documentation/general/guides/authorization-guide/#authorization-flows). One large benefit of this flow is that it doesn't require users to log into their Spotify account, minimizing the risk that a user will leave your app before logging in. It also means you don’t need to worry about redirecting the user back to the app (i.e. less code!). However, because the Client Credentials Flow does not require users to authorize access to their Spotify account, you won’t be able to retrieve any user information such as a users’ Spotify profile details, playback history, and playlists.

Now, let's get started...

### 1. Register a Spotify application to obtain a client ID and client secret
***

First, you will need to let Spotify know that you’re creating an app and want to use Spotify’s API.  To do this, go to your Spotify [Dashboard]( https://developer.spotify.com/documentation/general/guides/app-settings/), click on ‘Create an app’, and then follow the on-screen instructions. See this [doc](https://developer.spotify.com/documentation/general/guides/app-settings/) for more details. After you have registered your app, you will be provided with a client ID and client secret.

### 2. Store your client ID and client secret in environment variables
***

Next, create a file titled ‘.env’ in your program’s root directory and include the following information (replace the ‘X’s with your client ID and client secret from step 1):


```
REACT_APP_CLIENT_ID=XXXXXXXXXXXXXXXXXXXXX
REACT_APP_CLIENT_SECRET= XXXXXXXXXXXXXXXX
```

Then, in your React component, call your client secret and client ID with `process.env.REACT_REACT_APP_CLIENT_SECRET` and `process.env.APP_CLIENT_ID` respectively. Your component should look something like this:


```
import React, { Component } from 'react';

class SearchInput extends Component {

    static CLIENT_ID = process.env.REACT_APP_CLIENT_ID;
    static CLIENT_SECRET = process.env.REACT_APP_CLIENT_SECRET;

   // SOME CODE HERE

}

export default SearchInput;
```

### 3. Retrieve an access token from Spotify
***

To use Spotify’s API via the Client Credentials Flow, you need to exchange your client ID and client secret for an access token. We’ll be using a snippet of code from this [Client Credentials Flow example](https://github.com/spotify/web-api-auth-examples), created by former Spotify software engineer José M. Pérez.

```
import React, { Component } from 'react';

class SearchInput extends Component {

    static CLIENT_ID = process.env.REACT_APP_CLIENT_ID;
    static CLIENT_SECRET = process.env.REACT_APP_CLIENT_SECRET;
  
    state = { query: "" }
    
    accessSpotifyAPI = (query) => {
        
      const request = require('request')

      const authOptions = {
        url: 'https://accounts.spotify.com/api/token',
        headers: { 'Authorization': 'Basic ' + (new Buffer(SearchInput.CLIENT_ID + ':' + SearchInput.CLIENT_SECRET).toString('base64')) },
        form: { grant_type: 'client_credentials' },
        json: true
      };
        
      request.post(authOptions, (error, response, body) => {
        if (!error && response.statusCode === 200) {
          const token = body.access_token;

          // DO SOMETHING WITH THE TOKEN

        }
      })
    }

    handleChange = (e) => this.setState({ query: e.target.value })
    handleSubmit = (e) => {
      e.preventDefault();        
      this.accessSpotifyAPI(this.state.query)
      this.setState({ query: '' })
    }

    render() {
        return (
            <div>
                <h2>SEARCH FOR A SONG</h2>
                <form onSubmit={ this.handleSubmit }>
                    <input type="text" onChange={ this.handleChange } value={this.state.query} placeholder='Enter the name of a song, artist, or album' />
                    <input type="submit" value=”Submit” />
                </form>
            </div>
        )
    }
}

export default SearchInput;
```

I’m going to assume the reader has some basic knowledge of React so I won’t go into explaining `state` or what’s in `render`, since these pieces of code are quite standard in React.

What’s important to know is that the `accessSpotifyAPI` method is posting a request to Spotify’s Account Service server via node’s request module. Notice how the client ID and client secret are stored in `authOptions`, which is then included in the `request.post` method. This method includes a callback that returns a JSON object with your access token, which you need in order to use Spotify's Search API.

### 4. Access Spotify’s Search API
***

Let’s put the access token to use. For this, we’re going to send another request to Spotify’s server via the [Spotify Web API JS wrapper](https://github.com/JMPerez/spotify-web-api-js). This wrapper was developed by none other than José M. Pérez.

Run `npm install -S spotify-web-api-js` to install the wrapper via node.

Next, import the wrapper into your component.

```
import SpotifyWebApi from 'spotify-web-api-js';
```

Let’s now go back to where we left off in our SearchInput class. Right after we get our access token from Spotify’s server, we want to share the token with our wrapper, since it's actually our wrapper that will be interacting with Spotify’s API.

```
const spotify = new SpotifyWebApi();
spotify.setAccessToken(token)
```

Finally, we can use our wrapper to access Spotify’s Search API, like this:

```
spotify.searchTracks(query, { limit: 10 }).then(
(data) => {
    
		// DO SOMETHING WITH THE RESULTS!
		console.log(‘Search results’, data.tracks.items)
		
},
(err) => {
    console.error(err);
}
)
```

The `query` argument is whatever is stored in `state` when the form is submitted (i.e. the name of the song, author, or album). The option `{ limit: 10 }` specifies the number of tracks we would like Spotify’s Search API to return information on. Note that Spotify’s API limits the maximum number of songs to 50.

Here is the final code:

```
import React, { Component } from 'react';
import SpotifyWebApi from 'spotify-web-api-js';

class SearchInput extends Component {

    static CLIENT_ID = process.env.REACT_APP_CLIENT_ID;
    static CLIENT_SECRET = process.env.REACT_APP_CLIENT_SECRET;
  
    state = { query: "" }
    
    accessSpotifyAPI = (query) => {
        
      const request = require('request')      
      const authOptions = {
        url: 'https://accounts.spotify.com/api/token',
        headers: { 'Authorization': 'Basic ' + (new Buffer(SearchInput.CLIENT_ID + ':' +  SearchInput.CLIENT_SECRET).toString('base64')) },
        form: { grant_type: 'client_credentials' },
        json: true
      };
        
      request.post(authOptions, (error, response, body) => {
        if (!error && response.statusCode === 200) {
          
          const token = body.access_token;
          const spotify = new SpotifyWebApi();
          
          spotify.setAccessToken(token)
          spotify.searchTracks(query, { limit: 10 }).then(
            (data) => {

              // DO SOMETHING WITH THE RESULTS!
	            console.log(‘Search results’, data.tracks.items)

            },
            (err) => {
              console.error(err);
            }
          )
        }
      })
    }
  
    handleChange = (e) => this.setState({ query: e.target.value })
    handleSubmit = (e) => {
      e.preventDefault();        
      this.accessSpotifyAPI(this.state.query)
      this.setState({ query: '' })
    }    

    render() {
        return (
            <div>
                <h2>SEARCH FOR A SONG</h2>
                <form onSubmit={ this.handleSubmit }>
                    <input type="text" onChange={ this.handleChange } value={this.state.query} placeholder='Enter the name of a song, artist, or album' />
                    <input type="submit" value=”Submit” />
                </form>
            </div>
        )
    }
}

export default SearchInput;
```

That’s it! Using Spotify’s Client Credentials Flow and the Spotify Web API JS wrapper, you can retrieve information on a song without requiring users to log into their Spotify account.

### Resources
***

* [Spotify Search API](https://developer.spotify.com/documentation/web-api/reference/#category-search)
* [Spotify Web API JS Wrapper](https://github.com/JMPerez/spotify-web-api-js)
* [Spotify Client Credentials Flow Example](https://github.com/spotify/web-api-auth-examples/blob/master/client_credentials/app.js)
