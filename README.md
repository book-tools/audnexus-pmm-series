# Audnexus Plex-Meta-Manager Audiobook Series Collections

This is a guide on how to set up
[Plex-Meta-Manager](https://github.com/meisnate12/Plex-Meta-Manager) to
automatically generate collections from your audiobook series' that are
populated from the
[Audnexus metadata agent](https://github.com/djdembeck/Audnexus.bundle) for
Plex. This is done by generating collections from the "Mood" tags that Audnexus
adds to each book's metadata from each Audible series they are a part of.

Plex-Meta-Manager is "project that has been designed to ease the creation and
maintenance of metadata, collections, and playlists within a Plex Media Server".
There are other tools made with a similar purpose such as
[Plex-Auto-Collections](https://github.com/mza921/Plex-Auto-Collections) but
I've had the most success with PMM.

## 1. Setting Up Audnexus

First, you'll need to set the Audnexus metadata agent up using
[their official guide](https://github.com/djdembeck/Audnexus.bundle#-getting-started-).
During this process, you should uncheck the "Append authors as Mood tags"
option, unless you'd like a collection to be created for each author in addition
to each series. If you already have this agent set up and had previously checked
this option, you may run into issues with getting the author tags to be removed
on a metadata refresh.

## 2. Setting up Plex-Meta-Manager

Next, you'll need to set up Plex-Meta-Manager, using
[their official guide](https://metamanager.wiki/en/latest/home/installation.html).
Personally, I've only set up this tool using
[Docker](https://metamanager.wiki/en/latest/home/guides/docker.html)
(specifically
[unRAID](https://metamanager.wiki/en/latest/home/guides/unraid.html)), and as of
writing this, the `latest` tag is not compatible with generating collections
from the "mood" tag. Because of this, you'll need to pull the `nightly` version
instead of `latest` when installing the container.

The same goes for the
[local installation](https://metamanager.wiki/en/latest/home/guides/local.html)
instructions. When you get to the
[Retrieving the Plex-Meta-Manager code](https://metamanager.wiki/en/latest/home/guides/local.html#retrieving-the-plex-meta-manager-code)
section, you'll need to do an extra step to get the `nightly` branch.

```
git clone https://github.com/meisnate12/Plex-Meta-Manager
cd Plex-Meta-Manager
git checkout nightly
```

Or alternatively, if you just want to download the code as a zip file, use this
link:
https://github.com/meisnate12/Plex-Meta-Manager/archive/refs/heads/nightly.zip

Eventually this will be unnecessary, and I'll update this guide when the changes
in nightly are merged into the main branch.

In either setup (docker or manual), you'll eventually get to a point where
you'll need to set up your config files. First, I recommend copying the two YML
files included in this repo into your root config directory.

#### `config/config.yml`

```yml
libraries:
  REPLACE_WITH_AUDIOBOOK_LIBRARY_NAME:
    metadata_path:
      - file: config/audiobooks.yml
    operations:
      delete_collections:
        unconfigured: true
        less: 1

plex:
  url: REPLACE_WITH_LOCAL_PLEX_URL
  token: REPLACE_WITH_PLEX_TOKEN

tmdb:
  apikey: REPLACE_WITH_TMDB_API_KEY
```

#### `config/audiobooks.yml`

```yml
templates:
  audiobooks:
    builder_level: album # ensure each collection is made from an entire book's album, not individual tracks on it
    smart_filter:
      limit: 1000 # without a custom limit, the max books in a series will be cut off at 10.
      sort_by: title.asc # sort each series by sort title ascending, as the sort title for each book in a series (from the metadata agent) should keep them in order. without this, the books in the collection will be sorted by total plays.
      all:
        album_mood: <<value>>

dynamic_collections:
  Audiobook Series:
    type: album_mood
    title_format: <<key_name>>
    remove_prefix: 'Series: ' # remove the "Series: " prefix from each Series name added by Audnexus before creating a collection from it.
    # remove_suffix: ' Series' # some audible series names include the word "Series" at the end. If you'd like to remove that, uncomment this option.
    template: audiobooks
```

Then, you'll need to replace the variables in `config.yml` I left in `ALL_CAPS`
with their respective values:

1. `REPLACE_WITH_AUDIOBOOK_LIBRARY_NAME` - Replace this value with human
   readable name your audiobook library in Plex was created with. In my case
   it's simply "Audiobooks".
2. `REPLACE_WITH_LOCAL_PLEX_URL` - This should be replaced with the local URL
   your plex server can be found at, often just an IP address. In my case this
   is `http://http://192.168.1.2:32400`.
3. `REPLACE_WITH_PLEX_TOKEN` - For this part, you'll need to find your Plex
   authentication token following the
   [official guide on the Plex website](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/).
4. `REPLACE_WITH_TMDB_API_KEY` - While this setup will not require
   [TMDB](https://www.themoviedb.org/) in any way, it is still required for
   Plex-Meta-Manager to run. It is, however, very easy to obtain one following
   [PMM's guide for it](https://metamanager.wiki/en/latest/home/guides/local.html#getting-a-tmdb-api-key).

## Finishing Up

And that should be all you need! Once both config files are in the right place
with all the proper variables replaced, you can start up Plex-Meta-Manager to
see if everything is working properly. You will know it is if you see
collections being created and populated for each `Series: Series Name` mood tags
each of your books is tagged with.

The next step for me would be to set up PMM on a schedule so it always generates
new collections as new books are added to the server. You can do that by
checking out their
[Scheduling Guide](https://metamanager.wiki/en/latest/home/guides/scheduling.html).

If you have any questions, run into any issues getting this set up, or have any
suggestions to improve my config, feel free to leave an issue on this repo.
