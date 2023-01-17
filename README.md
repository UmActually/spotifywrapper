# spotifyatlas

### A pythonic wrapper for the Spotify web API.

By Leonardo - UmActually

`spotifyatlas` is a straightforward library meant to simplify the process of interacting with **Spotify's web API**. Whether you are trying to automate the process of **search queries**, or **modifying your playlists**, spotifyatlas has tools for your scripting (or webdev) needs, all in a clean, object-oriented style.

Most of the package's functionality is included in the ``SpotifyAPI`` class, which only needs to be initialized with the **credentials** of your client application. This codebase was originally used to retrieve track details for **Discord bots**, so most of the functions, as of now, revolve around playlists, albums, top tracks of artists, and whatnot.

## Basic Usage

_Refer to the [Installation](#installation) section for information on how to install the package with `pip`._

The first step to interact with the Spotify API is to register a new **application** in the **[Spotify for Developers](https://developer.spotify.com/dashboard/)** page. Worry not: this process is completely free for any Spotify user (with an account).

### Quick Start

With that out of the way, go ahead and initialize a new `SpotifyAPI` object with the credentials of your app (client ID and client secret):

```python
from spotifyatlas import SpotifyAPI
spoti = SpotifyAPI('<my-client-id>', '<my-client-secret>')
```

If you wish to retrieve the **tracks and/or details** of anything in Spotify, the universal `get()` method many times will get you covered. Try it by pasting the share link of your favorite playlist. It will return a `Result`: the playlist tracks are located in the `tracks` attribute.

```python
from spotifyatlas import SpotifyAPI
spoti = SpotifyAPI('<my-client-id>', '<my-client-secret>')
result = spoti.get('https://open.spotify.com/playlist/6xTnvRqIKptVfgcT8gN4Bb')
print(result.tracks)
# [Track('Goliath', 'The Mars Volta', '3bi3Ycf0ZlRHvSg0IxlMwM'), ... ]
```

A `Track` contains the `name`, `artist` and `id` of a song. Neatly list the contents of your playlist like this:

```python
for i, track in enumerate(result.tracks, 1):
    print(i, track.name, track.artist, sep=' - ')
# 1 - Goliath - The Mars Volta
# 2 - Juicy - 2005 Remaster - The Notorious B.I.G.
# 3 - O Peso da Cruz - Supercombo
# 4 - Count The People (feat. Jessie Reyez & T-Pain) - Jacob Collier
# ...
```

### Getting

The following methods offer the same functionality as `get()`, although more specific:

- `get_playlist()` for public playlists.

- `get_track()` for tracks.

- `get_artist()` for artists and their top 10 tracks in the US.

- `get_album()` for albums.

- `get_user()` for users. This returns a `UserResult`.

They all require the `url` or the ID of the element as the first argument.

### Searching

Not everything demands you having the link of the item at hand. To perform **searches**, you can use the following methods:

- `search()` to search normally, with the option to specify result types.

```python
result = spoti.search('ok human')
top_album_result = result.albums[0]
print(top_album_result.tracks)
# [Track('All My Favorite Songs', 'Weezer', '6zVhXpiYbJhLJWmLGV9k1r'), ... ]
```

- `im_feeling_lucky()` if you know in advance exactly what you are looking for. It is essentially the same as `search()` but returns directly the top `Result` of the specified type.

```python
result = spoti.im_feeling_lucky('quevedo biza', spotifyatlas.Type.TRACK)
print(result)
# Quevedo: Bzrp Music Sessions, Vol. 52 - Bizarrap
```

- The standalone function `spotifyatlas.advanced_search()` can generate a more [powerful search query](https://support.spotify.com/us/article/search/) that the user can then pass to either of the search methods (or even paste to their actual Spotify client).

```python
from spotifyatlas import advanced_search

overkill_query = advanced_search(
    'juanes',
    album='metallica',
    year=2021,
    genre=spotifyatlas.Genre.ROCK
)

result = spoti.im_feeling_lucky(overkill_query, spotifyatlas.Type.TRACK)
print(result)
# Enter Sandman - Juanes
```

### User Functionality

These other methods require **user consent**, and thus will result in the **browser** opening for the authorization of your application to act on behalf of the user:

- `get_me()` for details of your own profile.

- `get_private_playlist()` for private playlists you own.

- `create_playlist()` to make a new empty playlist in your library.

- `add_to_playlist()` to add a batch of `Track`s to a playlist.

- `create_playlist_copy()` to duplicate a playlist.

- `clear_playlist()` to remove all the contents of a playlist.

- `rearrange_playlist()` to change the position of a range of tracks.

_Note: authorizing the application in the Spotify authorization page requires a **redirection** page to go to. By default, this library will temporarily **host a local page** on http://localhost:8000. Thus, you **will need to add this URL** to the allowed redirection URLs on the dashboard of your application in the **[Spotify for Developers](https://developer.spotify.com/dashboard/)** site._

The complete list of parameters/arguments of a function can be found in its documentation.

---

## Installation

To install spotifyatlas, use **pip** in the terminal:

**Windows**
```commandline
pip install spotifyatlas
```

**macOS / Linux**
```commandline
python3 -m pip install spotifyatlas
```

---

## More Examples

For the inquisitive user, here are some more code examples out the top of my head:

### 1. Rearrange the tracks of a playlist by artist, alphabetically

```python
from spotifyatlas import SpotifyAPI, Track


def artist_sort_key(_track: Track) -> str:
    return _track.artist.lower()


MY_PLAYLIST = '<my-playlist-link>'
spoti = SpotifyAPI('<my-client-id>', '<my-client-secret>')

# make_copy makes a backup of the playlist before removing its contents
result = spoti.clear_playlist(MY_PLAYLIST, make_copy=True)

# result is the state of the playlist before being cleared
tracks = result.tracks
tracks.sort(key=artist_sort_key)

spoti.add_to_playlist(MY_PLAYLIST, tracks)
```

### 2. Find the songs that two playlists have in common

```python
from spotifyatlas import SpotifyAPI

MY_PLAYLIST = '<my-playlist-link>'
MY_FRIENDS_PLAYLIST = '<my-friends-playlist-link>'

spoti = SpotifyAPI('<my-client-id>', '<my-client-secret>')

playlist1 = spoti.get(MY_PLAYLIST).tracks
playlist2 = spoti.get(MY_FRIENDS_PLAYLIST).tracks

# Set theory!!!
blend = set(playlist1).intersection(set(playlist2))
for i, track in enumerate(blend, 1):
    print(i, track, sep=' - ')
```
