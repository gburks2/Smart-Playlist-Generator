import tkinter as tk
from collections import defaultdict
from tkinter import ttk, scrolledtext, filedialog, messagebox
import threading
import sys
from io import StringIO
import random
import main as pg


# Redirect console output to the UI
class RedirectOutput:
    def __init__(self, text_widget):
        self.output = text_widget
        self.buffer = ""

    def write(self, string):
        self.buffer += string
        self.output.insert(tk.END, string)
        self.output.see(tk.END)  # Auto-scroll

    def flush(self):
        pass


# Initial screen to select playlist generation method
class StartupScreen:
    def __init__(self, root):
        self.root = root
        self.root.title("Choose Your Playlist")
        self.root.geometry("400x300")

        # Color scheme
        bg = "#121212"
        green = "#1DB954"
        text = "#FFFFFF"

        self.root.configure(bg=bg)

        # Setup styling
        style = ttk.Style()
        style.configure("TFrame", background=bg)
        style.configure("TLabel", background=bg, foreground=text)
        style.configure("TButton", background=green, foreground="black", padding=5)

        # Selected algorithm
        self.algorithm = tk.StringVar(value="tags")

        # Main container
        main = ttk.Frame(self.root, padding="20")
        main.pack(fill=tk.BOTH, expand=True)

        # Heading
        welcome = ttk.Label(main, text="Music Playlist Generator", font=("Arial", 18, "bold"))
        welcome.pack(pady=(20, 10))

        subtitle = ttk.Label(main, text="How would you like to build your playlist?", font=("Arial", 12))
        subtitle.pack(pady=(0, 30))

        # Options
        option_frame = ttk.Frame(main)
        option_frame.pack(fill=tk.X, padx=20, pady=10)

        # Tags option button
        tags_btn = ttk.Button(option_frame, text="By Music Tags",
                              command=lambda: self.start_app("tags"),
                              width=20)
        tags_btn.pack(pady=10)

        tags_desc = ttk.Label(option_frame, text="Find music with similar styles and genres")
        tags_desc.pack(pady=(0, 15))

        # Popularity option button
        pop_btn = ttk.Button(option_frame, text="By Popularity",
                             command=lambda: self.start_app("popularity"),
                             width=20)
        pop_btn.pack(pady=10)

        pop_desc = ttk.Label(option_frame, text="Mix popular tracks with hidden gems")
        pop_desc.pack(pady=(0, 15))

    def start_app(self, algorithm):
        self.algorithm = algorithm
        self.root.destroy()


# Main application after method selection
class PlaylistApp:
    def __init__(self, root, algorithm="tags"):
        self.root = root
        self.root.title("Music Playlist Generator")
        self.root.geometry("750x550")

        # App variables
        self.track_query = tk.StringVar()
        self.artist_query = tk.StringVar()
        self.playlist_size = tk.IntVar(value=15)
        self.algorithm = algorithm  # Set from startup selection
        self.search_mode = "track"

        # Data storage
        self.track_results = []
        self.artist_results = []
        self.track_objects = []
        self.artist_objects = []
        self.playlist = []

        self.create_ui()

    def create_ui(self):
        # Color scheme
        bg = "#121212"
        green = "#1DB954"
        text = "#FFFFFF"
        alt_bg = "#282828"

        self.root.configure(bg=bg)

        # Configure widget styles
        style = ttk.Style()
        style.configure("TFrame", background=bg)
        style.configure("TLabel", background=bg, foreground=text)
        style.configure("TButton", background=green, foreground="black", padding=5)
        style.configure("TLabelframe", background=bg)
        style.configure("TLabelframe.Label", background=bg, foreground=text)
        style.configure("TCombobox", fieldbackground=alt_bg, foreground=text)

        # Main container
        main = ttk.Frame(self.root, padding="10")
        main.pack(fill=tk.BOTH, expand=True)

        # Display header with selected mode
        mode_text = "Music Tags" if self.algorithm == "tags" else "Popularity"
        header = ttk.Label(main, text=f"Choose Your Playlist - {mode_text} Mode", font=("Arial", 16, "bold"))
        header.pack(pady=(0, 15))

        # Search section
        search_frame = ttk.LabelFrame(main, text="Search", padding="10")
        search_frame.pack(fill=tk.X, padx=10, pady=5)

        # Track search row
        ttk.Label(search_frame, text="Track:").grid(row=0, column=0, padx=5, pady=5)
        ttk.Entry(search_frame, textvariable=self.track_query, width=30).grid(row=0, column=1, padx=5, pady=5)
        ttk.Button(search_frame, text="Search", command=self.search_track).grid(row=0, column=2, padx=5, pady=5)
        self.track_label = ttk.Label(search_frame, text="None selected")
        self.track_label.grid(row=0, column=3, padx=5, pady=5)

        # Artist search row
        ttk.Label(search_frame, text="Artist:").grid(row=1, column=0, padx=5, pady=5)
        ttk.Entry(search_frame, textvariable=self.artist_query, width=30).grid(row=1, column=1, padx=5, pady=5)
        ttk.Button(search_frame, text="Search", command=self.search_artist).grid(row=1, column=2, padx=5, pady=5)
        self.artist_label = ttk.Label(search_frame, text="None selected")
        self.artist_label.grid(row=1, column=3, padx=5, pady=5)

        # Results section for search results
        results_frame = ttk.LabelFrame(main, text="Results", padding="10")
        results_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

        # Listbox to display search results
        self.results_box = tk.Listbox(results_frame, height=6, width=60, bg=alt_bg, fg=text, selectbackground=green)
        self.results_box.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)

        # Select button for search results
        select_btn = ttk.Button(results_frame, text="Select", command=self.select_result)
        select_btn.pack(pady=5)

        # Options section
        options_frame = ttk.LabelFrame(main, text="Options", padding="10")
        options_frame.pack(fill=tk.X, padx=10, pady=5)

        # Playlist size control
        size_frame = ttk.Frame(options_frame)
        size_frame.pack(side=tk.LEFT, padx=20, pady=5)

        ttk.Label(size_frame, text="Playlist Size:").pack(side=tk.LEFT, padx=5)
        ttk.Spinbox(size_frame, from_=5, to=50, textvariable=self.playlist_size, width=5).pack(side=tk.LEFT, padx=5)

        # Generate button - disabled until track and artist are selected
        self.generate_btn = ttk.Button(options_frame, text="Generate Playlist",
                                       command=self.generate_playlist, state=tk.DISABLED)
        self.generate_btn.pack(side=tk.LEFT, padx=20)

        # Save button - disabled until playlist is generated
        self.save_btn = ttk.Button(options_frame, text="Save Playlist",
                                   command=self.save_playlist, state=tk.DISABLED)
        self.save_btn.pack(side=tk.LEFT, padx=5)

        # Console output area
        console_frame = ttk.LabelFrame(main, text="Musical Discovery Lab", padding="10")
        console_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

        # Text widget for progress output
        self.output_text = scrolledtext.ScrolledText(console_frame, height=6, bg=alt_bg, fg=text)
        self.output_text.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)

        # Redirect stdout to our text widget
        self.stdout_redirect = RedirectOutput(self.output_text)
        sys.stdout = self.stdout_redirect

        # Status bar at bottom
        self.status = tk.StringVar(value="Ready")
        status_bar = ttk.Label(self.root, textvariable=self.status, relief=tk.SUNKEN, anchor=tk.W)
        status_bar.pack(side=tk.BOTTOM, fill=tk.X)

    # Track search function
    def search_track(self):
        if not self.track_query.get().strip():
            return

        self.status.set("Searching for tracks...")
        self.output_text.delete('1.0', tk.END)
        self.results_box.delete(0, tk.END)
        self.search_mode = "track"

        # Run search in background thread
        threading.Thread(target=self._do_track_search, daemon=True).start()

    # Background thread for track search
    def _do_track_search(self):
        try:
            old_stdout = sys.stdout
            capture = StringIO()
            sys.stdout = capture

            self.track_results = []
            self.track_objects = []

            # Call API to find tracks
            results = pg.network.search_for_track("", self.track_query.get()).get_next_page()

            for i, track in enumerate(results[:10]):
                artist_name = track.get_artist().get_name()
                track_name = track.get_title()
                self.track_results.append(f"[{i}] {track_name} by {artist_name}")
                self.track_objects.append(track)

            sys.stdout = old_stdout
            self.root.after(0, self._update_track_results)
        except Exception as e:
            sys.stdout = old_stdout
            print(f"Error during search: {str(e)}")
            self.root.after(0, lambda: self.status.set("Search failed!"))

    # Update UI with track results
    def _update_track_results(self):
        for result in self.track_results:
            self.results_box.insert(tk.END, result)
        self.status.set(f"Found {len(self.track_results)} tracks")

    # Artist search function
    def search_artist(self):
        if not self.artist_query.get().strip():
            return

        self.status.set("Searching for artists...")
        self.output_text.delete('1.0', tk.END)
        self.results_box.delete(0, tk.END)
        self.search_mode = "artist"

        # Run search in background thread
        threading.Thread(target=self._do_artist_search, daemon=True).start()

    # Background thread for artist search
    def _do_artist_search(self):
        try:
            old_stdout = sys.stdout
            capture = StringIO()
            sys.stdout = capture

            self.artist_results = []
            self.artist_objects = []

            # Call API to find artists
            results = pg.network.search_for_artist(self.artist_query.get()).get_next_page()

            for i, artist in enumerate(results[:10]):
                name = artist.get_name()
                self.artist_results.append(f"[{i}] {name}")
                self.artist_objects.append(artist)

            sys.stdout = old_stdout
            self.root.after(0, self._update_artist_results)
        except Exception as e:
            sys.stdout = old_stdout
            print(f"Error during search: {str(e)}")
            self.root.after(0, lambda: self.status.set("Search failed!"))

    # Update UI with artist results
    def _update_artist_results(self):
        for result in self.artist_results:
            self.results_box.insert(tk.END, result)
        self.status.set(f"Found {len(self.artist_results)} artists")

    # Handle selection from results list
    def select_result(self):
        selected = self.results_box.curselection()
        if not selected:
            return

        idx = selected[0]

        # Handle track selection
        if self.search_mode == "track":
            selected_track = self.track_objects[idx]
            track = pg.network.get_track(
                selected_track.get_artist().get_name(),
                selected_track.get_title()
            )
            title = track.get_title()
            artist = track.get_artist().get_name()

            self.selected_track = track
            self.selected_track_obj = pg.Song(
                ID=f"{artist} - {title}",
                artist=artist,
                name=title,
                image=pg.TryGetCover(track),
                tags=None
            )
            self.track_label.config(text=f"{title}")

        # Handle artist selection
        elif self.search_mode == "artist":
            selected_artist = self.artist_objects[idx]
            artist = pg.network.get_artist(selected_artist.get_name())

            self.selected_artist = artist
            self.selected_artist_obj = pg.Artist(
                ID=artist.get_mbid() or artist.get_name(),
                name=artist.get_name(),
                image=pg.TryGetImage(artist),
                tags=None
            )
            self.artist_label.config(text=f"{artist.get_name()}")

        # Enable generate button if both track and artist are selected
        if hasattr(self, 'selected_track') and hasattr(self, 'selected_artist'):
            self.generate_btn.config(state=tk.NORMAL)

    # Generate playlist based on selections
    def generate_playlist(self):
        if not hasattr(self, 'selected_track') or not hasattr(self, 'selected_artist'):
            return

        self.status.set("Working on it...")
        self.generate_btn.config(state=tk.DISABLED)
        self.output_text.delete('1.0', tk.END)

        # Run generation in background thread
        threading.Thread(target=self._do_generate_playlist, daemon=True).start()

    # Fetch tags for track/artist
    def _get_tags(self, item, limit=10):
        tags = set()
        try:
            item_tags = item.get_top_tags(limit=limit)
            for tag in item_tags:
                tags.add(tag.item.get_name().lower())
        except:
            pass
        return tags

    # Tags-based playlist generation algorithm
    def _generate_playlist_by_tags(self, track_pool, size):
        seed_tags = set()

        # Get tags from selected track and artist
        track_tags = self._get_tags(self.selected_track)
        seed_tags.update(track_tags)

        artist_tags = self._get_tags(self.selected_artist)
        seed_tags.update(artist_tags)

        print(f"Found {len(seed_tags)} tags from your selections")
        if seed_tags:
            print(f"Tags: {', '.join(seed_tags)}")

        if not seed_tags:
            print("No tags found, using random selection")
            return random.sample(track_pool, min(size, len(track_pool)))

        # Get tags for all potential tracks
        for track in track_pool:
            if not hasattr(track, 'tags') or not track.tags:
                try:
                    lastfm_track = pg.network.get_track(track.artist, track.name)
                    track_tags = self._get_tags(lastfm_track, limit=5)
                    track.tags = list(track_tags)
                except:
                    track.tags = []

        # Score tracks by tag similarity
        scored_tracks = []
        for track in track_pool:
            if track.name == self.selected_track_obj.name and track.artist == self.selected_track_obj.artist:
                continue

            track_tags = set(track.tags) if hasattr(track, 'tags') and track.tags else set()

            # Calculate Jaccard similarity between track tags and seed tags
            if track_tags and seed_tags:
                intersection = len(track_tags.intersection(seed_tags))
                union = len(track_tags.union(seed_tags))
                similarity = intersection / union if union > 0 else 0
            else:
                similarity = 0

            scored_tracks.append((track, similarity))

        # Sort by similarity score
        scored_tracks.sort(key=lambda x: x[1], reverse=True)

        result = [track for track, _ in scored_tracks[:size]]

        # Add random tracks if needed to reach desired size
        if len(result) < size:
            remaining = size - len(result)
            remaining_pool = [t for t in track_pool if t not in result]
            if remaining_pool:
                result.extend(random.sample(remaining_pool, min(remaining, len(remaining_pool))))

        print(f"Generated {len(result)} tracks based on music tags")
        return result

    # Popularity-based playlist generation algorithm
    def _generate_playlist_by_popularity(self, track_pool, size):
        print("Analyzing track popularity...")

        scored_tracks = []

        # Score tracks by popularity metrics
        for track in track_pool:
            if track.name == self.selected_track_obj.name and track.artist == self.selected_track_obj.artist:
                continue

            try:
                lastfm_track = pg.network.get_track(track.artist, track.name)

                try:
                    playcount = int(lastfm_track.get_playcount())
                except:
                    playcount = 0

                try:
                    listeners = int(lastfm_track.get_listener_count())
                except:
                    listeners = 0

                popularity = playcount + (listeners * 10)

            except:
                popularity = random.randint(1, 1000)

            scored_tracks.append((track, popularity))

        # Sort by popularity
        scored_tracks.sort(key=lambda x: x[1], reverse=True)

        # Take top 60% popular tracks
        top_count = int(size * 0.6)
        top_tracks = [track for track, _ in scored_tracks[:top_count]]

        # Mix in some random tracks for diversity
        remaining_count = size - len(top_tracks)
        remaining_pool = [track for track, _ in scored_tracks[top_count:]]

        if remaining_pool and remaining_count > 0:
            random_tracks = random.sample(remaining_pool, min(remaining_count, len(remaining_pool)))
        else:
            random_tracks = []

        result = top_tracks + random_tracks

        print(f"Generated {len(result)} tracks based on popularity")
        return result

    # Background thread for playlist generation
    def _do_generate_playlist(self):
        try:
            method = "Music Tags" if self.algorithm == "tags" else "Popularity"
            print(f"Building playlist based on {self.selected_track_obj.name} & {self.selected_artist_obj.name}")

            size = self.playlist_size.get()

            if self.algorithm == "tags":
                track_pool = pg.generateTrackPool_tags(40, self.selected_track, self.selected_artist)
            else:
                buckets = pg.generate_track_pool_popularity(self.selected_track, self.selected_artist)

                track_pool = [song for _, song_list in buckets for song in song_list]

                track_pool = track_pool[:40]

            print(f"Found {len(track_pool)} potential tracks!")

            if self.algorithm == "tags":
                self.playlist = self._generate_playlist_by_tags(track_pool, size)
            else:
                self.playlist = self._generate_playlist_by_popularity(track_pool, size)

            title = f"Playlist inspired by {self.selected_track_obj.name} and {self.selected_artist_obj.name}"
            pg.display_playlist(self.playlist, title)

            self.root.after(0, self._update_after_generation)
        except Exception as e:
            print(f"Error: {str(e)}")
            import traceback
            traceback.print_exc()
            self.root.after(0, lambda: self.status.set("Error: Playlist generation failed"))
            self.root.after(0, lambda: self.generate_btn.config(state=tk.NORMAL))

    # Update UI after playlist generation
    def _update_after_generation(self):
        self.status.set("Done! Playlist ready.")
        self.generate_btn.config(state=tk.NORMAL)
        self.save_btn.config(state=tk.NORMAL)

    # Save playlist to file
    def save_playlist(self):
        if not self.playlist:
            return

        filename = filedialog.asksaveasfilename(
            defaultextension=".txt",
            filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
        )

        if not filename:
            return

        try:
            method = "Music Tags" if self.algorithm == "tags" else "Popularity"

            with open(filename, 'w', encoding='utf-8') as f:
                f.write(f"Playlist based on {self.selected_track_obj.name} and {self.selected_artist_obj.name}\n")
                f.write(f"Generated using {method} matching\n")
                f.write("-" * 50 + "\n\n")

                for i, track in enumerate(self.playlist):
                    f.write(f"{i + 1}. {track.name} by {track.artist}\n")
                    if track.tags:
                        f.write(f"   Tags: {', '.join(track.tags)}\n")
                    f.write("\n")

            self.status.set(f"Playlist saved to {filename}")
        except Exception as e:
            self.status.set(f"Error saving file: {e}")


# Main app entry point
def main():
    # First show the startup screen to choose method
    startup = tk.Tk()
    startup_app = StartupScreen(startup)
    startup.mainloop()

    # Get the algorithm choice from startup screen
    algorithm = startup_app.algorithm

    root = tk.Tk()
    app = PlaylistApp(root, algorithm)
    root.mainloop()

if __name__ == "__main__":
    main()

import pylast
import math
import random
import time
from IPython.display import clear_output
from typing import Dict, List, Tuple

API_KEY = "7008b5589d58885ace36d3221e86cbd0"
API_SECRET = "a3f47c877288ee66e42a15bc603cffb6"
network = pylast.LastFMNetwork(api_key=API_KEY, api_secret=API_SECRET)


class Artist:
    # tags -> top three tags
    def __init__(self, ID, name, image, tags=None):
        self.id = ID
        self.name = name
        self.image = image
        self.tags = tags


class Song:
    # image -> cover image
    # tags -> top three tags
    def __init__(self, ID, artist, name, image, tags=None, playcount = 0):
        self.playcount = None
        self.id = ID
        self.artist = artist
        self.name = name
        self.image = image
        self.tags = tags
        self.playcount = playcount


def TryGetCover(track: pylast.Track):
    try:
        return track.get_cover_image()
    except Exception:
        return None


def TryGetImage(artist: pylast.Artist):
    try:
        return artist.get_image()
    except Exception:
        return None


def TryGetTags(track: pylast.Track):
    try:
        return track.get_top_tags()
    except Exception:
        return None


def initialQuery(type):
    query = input(f"Enter search query for favorite {type}: ")

    itemIDs = []
    itemText = []
    itemObjects = []
    count = 0
    # Queries database, then displays
    if type == "track":
        searchResults = network.search_for_track("", query).get_next_page()
        for item in searchResults[:10]:
            artistName = item.get_artist().get_name()
            trackName = item.get_title()
            itemText.append(f"[{count}]: {trackName} by {artistName}")
            itemIDs.append(f"{artistName} - {trackName}")
            itemObjects.append(item)
            count += 1

    elif type == "artist":
        searchResults = network.search_for_artist(query).get_next_page()
        for item in searchResults[:10]:
            artistName = item.get_name()
            itemText.append(f"[{count}]: {artistName}")
            itemIDs.append(artistName)
            itemObjects.append(item)
            count += 1
    else:
        return "Invalid query"

    print("-------------RESULTS-------------")
    for entry in itemText:
        print(entry)

    while True:
        try:
            index = int(input(f"Pick a {type} by index (0 to {count - 1}): "))
            if 0 <= index < count:
                break
            print("Invalid index.")
        except ValueError:
            print("Enter a number.")

    selected = itemObjects[index]

    if type == "track":
        track = network.get_track(selected.get_artist().get_name(), selected.get_title())
        title = track.get_title()
        artistId = track.get_artist().get_mbid()
        artistName = track.get_artist().get_name()
        return (track, Song(
            ID=f"{artistName} - {title}",
            artist=artistName,
            name=title,
            image=TryGetCover(track),
            tags=None
        ))

    elif type == "artist":
        artist = network.get_artist(selected.get_name())
        id = artist.get_mbid()
        return (artist, Artist(
            ID=id,
            name=artist.get_name(),
            image=TryGetImage(artist),
            tags=None
        ))


# trackSeed,artistSeed are pylast objects
# I know that just calling get_recommended on like 400 tracks is cheating, so I want to create a track pool with varying sources
# 1/4 of the tracks will be generated from lastFM's recommendation algorithm (simple: quarter cheating)
# 1/4 of the tracks will be generated from related artists
# Remaining tracks generated from the highest frequency tags of the tracks form the last three
# get some random artists -> get random albums -> get random songs from each

# After, pad with some random songs

# outputs custom objects not pylast objects
def generateTrackPool_tags(count: int, trackSeed: pylast.Track, artistSeed: pylast.Artist = None):
    # I found that there's some edge cases that arise when you don't generate enough tracks, so impose some minimum
    if (count < 20):
        raise ValueError("Count needs to be >= 20")
        return 0
    # Throttles execution to not be caught by rate limitter
    timeRateLimit = count / 10

    # generate seed integers to get varying track amounts per step
    x = 0
    y = 0
    z = 0

    while (True):
        # SYSTEM PARAMETERS
        sigma = count * (1 / 8)
        mean = count * 1 / 4
        x = math.floor(random.gauss(mean, sigma))
        y = math.floor(random.gauss(mean, sigma))
        z = count - x - y
        if (x > 0 and y > 0 and z > 0):
            break

    pool = set()  # Filled with Song class from this code
    uniqueTags = defaultdict(int)
    uniqueArtists = set()  # Filled with Artist class from this code

    # Adding seed artist
    uniqueArtists.add(
        Artist(ID=artistSeed.get_mbid(), name=artistSeed.get_name(), image=TryGetImage(artistSeed), tags=None))

    # Bias the tags to the seed track -> Throughout this code, I use that lastFM api returns the tags from most frequent to least
    seedTags = trackSeed.get_top_tags()[:3]

    for tt in seedTags:
        # Picked count/5 arbitrarily, but I think this should make sure step 3 uses the best tags
        tag = tt.item
        uniqueTags[tag.get_name()] += math.floor(count / 5)

    # Step one: generate x similar tracks TO seed track doing BFS-kinda thing
    tempTrack = trackSeed
    done = False
    while not done:
        try:
            similarTracks = tempTrack.get_similar(limit=x)
            if not similarTracks:
                print("⚠️ No similar tracks found, skipping step 1.")
                break
        except pylast.WSError:
            print("Similar tracks not found, shutting down.")
            return
        update = True
        for similar in similarTracks:

            track = similar.item  # pylist.track object

            if update:
                tempTrack = track
                update = False

            tags = TryGetTags(track)

            # Edge case where song doens't have tags
            if (tags == None):
                pool.add(Song(
                    ID=track.get_mbid(),
                    artist=track.get_artist().get_name(),
                    name=track.get_name(),
                    image=TryGetImage(track),
                    tags=None,
                    playcount=track.get_playcount()
                ))
            else:

                firstTags = [l.item for l in tags[:3]]
                for u in firstTags:
                    uniqueTags[u.get_name()] += 1

                pool.add(Song(
                    ID=track.get_mbid(),
                    artist=track.get_artist().get_name(),
                    name=track.get_name(),
                    image=TryGetImage(track),
                    tags=firstTags,
                    playcount=track.get_playcount()
                ))
            if (len(pool) == x - 1):
                done = True
                break
    # throttle for rate limits
    print("--->Step 1 done<---")
    time.sleep(timeRateLimit)

    # Step two: generate y similar tracks to seed album, BFS kind of thing again
    done = False
    tempArtist = artistSeed
    tempCount = 0

    while not done:
        similarArtists = tempArtist.get_similar(limit=3)
        update = True
        for similar in similarArtists:
            artist = similar.item  # pylast.Artist object

            if update:
                tempArtist = artist
                update = False

            uniqueArtists.add(Artist(
                ID=artist.get_mbid(),
                name=artist.get_name(),
                image=TryGetImage(artist),
                tags=None
            ))

            albums = artist.get_top_albums(limit=2)

            for top in albums:
                album = top.item

                # Accounting for edge case of track being a single
                try:
                    tracks = album.get_tracks()
                except pylast.WSError:
                    continue

                randomTracks = [t for t in tracks if random.choice([True, False])]  # <- from stack exchange
                for track in randomTracks:
                    # Theres some tracks that have bad database parameters for some reason, so wrap in try catch (bad I know)
                    try:
                        # Some errors from get_top_tags being blank, idk why pylast works like this but it does whatever
                        tags = TryGetTags(track)
                        if (tags == None):
                            pool.add(Song(
                                ID=track.get_mbid(),
                                artist=track.get_artist().get_name(),
                                name=track.get_name(),
                                image=TryGetImage(track),
                                tags=None,
                                playcount=track.get_playcount()
                            ))
                        else:
                            firstTags = [l.item for l in tags[:3]]
                            for u in firstTags:
                                uniqueTags[u.get_name()] += 1

                            pool.add(Song(
                                ID=track.get_mbid(),
                                artist=track.get_artist().get_name(),
                                name=track.get_name(),
                                image=TryGetImage(track),
                                tags=firstTags,
                                playcount=track.get_playcount()
                            ))

                        tempCount += 1
                        if tempCount == y:
                            done = True
                            break
                    except pylast.WSError:
                        continue
                if done:
                    break
            if done:
                break
    print("--->Step 2 done<---")
    time.sleep(timeRateLimit)
    # Step three -> Run through tags generate floor(z/3) for each of the top tags
    j = math.floor(z / 3)
    for i in range(0, 3):
        # Take the max frequency tag, then find songs as usual, then remove it from the dictionary when done
        maxTagName = max(uniqueTags, key=uniqueTags.get)  # <- from stack exchange

        # Loop count values, count tracks how many songs have been added
        # previousLength tracks the past length of pool to make sure the values are added successfully
        tempCount = 0
        previousLength = len(pool)

        tag = network.get_tag(maxTagName)
        similarTracks = tag.get_top_tracks(limit=j + 10)
        for similar in similarTracks:
            track = similar.item  # pylist.track object
            # Don't add tags back to uniqueTags (not the point here)
            tags = track.get_top_tags()
            pool.add(Song(
                ID=track.get_mbid(),
                artist=track.get_artist().get_name(),
                name=track.get_name(),
                image=TryGetImage(track),
                tags=[l.item.get_name() for l in tags[:3]],
                playcount=track.get_playcount()
            ))
            # Value added successfully
            if (len(pool) == previousLength + tempCount + 1):
                tempCount += 1
            if (tempCount == j):
                break

                # Then remove top frequency item
        del uniqueTags[maxTagName]
        time.sleep(timeRateLimit / 3)

    print("--->Step 3 done<---")
    return pool


# Algorithm to generate a diverse but cohesive playlist from our track pool
def generate_playlist_tags(track_pool, num_tracks):
    """Generate a playlist from the track pool based on tags and variety."""
    # If we have fewer tracks than requested, just return what we have
    if len(track_pool) <= num_tracks:
        return list(track_pool)

    available_tracks = list(track_pool)

    all_tags = {}
    for track in available_tracks:
        if track.tags:
            for tag in track.tags:
                if tag in all_tags:
                    all_tags[tag] += 1
                else:
                    all_tags[tag] = 1

    # Create a playlist with diverse tags
    playlist = []
    used_tags = set()

    # Start with a few random tracks to seed the playlist and create initial variety
    seed_size = min(3, len(available_tracks))
    seed_tracks = random.sample(available_tracks, seed_size)

    for track in seed_tracks:
        playlist.append(track)
        available_tracks.remove(track)
        if track.tags:
            used_tags.update(track.tags)

    while len(playlist) < num_tracks and available_tracks:
        # Score each track based on how much variety it adds
        best_track = None
        best_score = -1

        for track in available_tracks:
            score = 0
            if track.tags:
                # Higher score for tags not already in playlist
                # This prioritizes musical diversity
                for tag in track.tags:
                    if tag not in used_tags:
                        score += 3
                    else:
                        score += 1

                        # Add some randomness to avoid predictability
            # This ensures different playlists each time
            score += random.random()

            if score > best_score:
                best_score = score
                best_track = track

        if best_track:
            playlist.append(best_track)
            available_tracks.remove(best_track)
            if best_track.tags:
                used_tags.update(best_track.tags)
        else:
            # Fallback: add a random track if scoring fails
            track = random.choice(available_tracks)
            playlist.append(track)
            available_tracks.remove(track)

    return playlist


# Function to display the playlist in a user-friendly format
def display_playlist(playlist, title="Your Personalized Playlist"):
    """Display the generated playlist with track info and tags."""
    print("\n" + "=" * 60)
    print(f" {title} ".center(60, "="))
    print("=" * 60)

    for i, track in enumerate(playlist):
        print(f"\n{i + 1}. {track.name} by {track.artist}")
        if track.tags:
            print(f"   Tags: {', '.join(track.tags)}")

    print("\n" + "=" * 60)


def generate_track_pool_popularity(track_seed: pylast.Track, artist_seed: pylast.Artist) -> List[
    Tuple[int, List[Song]]]:
    from pylast import Track, Artist
    def generate_pool(seed):
        popularity_map: Dict[int, List[Song]] = {}
        if isinstance(seed, Track):
            similar = seed.get_similar(limit=10)
            for i in similar:
                current = i.item
                popularity_score = current.get_listener_count() or 0
                song = Song(
                    ID=current.get_mbid(),
                    artist=current.get_artist().get_name(),
                    name=current.get_title(),
                    image=TryGetCover(current),
                    tags=None
                )
                popularity_map.setdefault(popularity_score, []).append(song)

                # Throttle API calls to avoid rate limiting
                time.sleep(1)
        elif isinstance(seed, Artist):
            similar = seed.get_similar(limit=10)
            for i in similar:
                current = i.item
                popularity_score = current.get_listener_count() or 0

                for track in current.get_top_tracks(limit=5):
                    t = track.item
                    song = Song(
                        ID=t.get_mbid(),
                        artist=t.get_artist().get_name(),
                        name=t.get_title(),
                        image=TryGetCover(t),
                        tags=None
                    )
                    popularity_map.setdefault(popularity_score, []).append(song)
                # Throttle API calls to avoid rate limiting
                time.sleep(1)
        return popularity_map

    similar_tracks = generate_pool(track_seed)
    similar_artist = generate_pool(artist_seed)

    tempT = sorted(similar_tracks.items())
    tempA = sorted(similar_artist.items())
    scores = tempT + tempA
    return scores


def generate_playlist_popularity(scores: List[Tuple[int, List[Song]]], size: int) -> List[Song]:
    split_point = int(len(scores) * 0.30)
    popular_scores = scores[len(scores) - split_point:]
    remaining_scores = scores[:len(scores) - split_point]

    used = set()

    def insert_song(bucket):
        while True:
            popularity_score, songs = random.choice(bucket)
            song = random.choice(songs)
            if song not in used:
                used.add(song.id)
                return song

    playlist: List[Song] = []
    for it in range(int(size * 0.70)):
        playlist.append(insert_song(popular_scores))
    for it in range(int(size * 0.30)):
        playlist.append(insert_song(remaining_scores))

    random.shuffle(playlist)
    return playlist


def main():
    print("🎵 Welcome to the Super Auto Playlist 🎵")
    print("\nThis program will generate a personalized playlist based on:")
    print("  - A track you like")
    print("  - An artist you like")

    # Get seed track and artist from user
    (t, tempObjectT) = initialQuery("track")
    (a, tempObjectA) = initialQuery("artist")

    print(f"\nGenerating recommendations based on:")
    print(f"Track: {tempObjectT.name}")
    print(f"Artist: {tempObjectA.name}")
    print("\nWould you like us to build your track pool based off of tags or popularity?")
    choice = int(input("1 = Tags\n2 = Popularity\n"))
    print("\nPlease wait while we build your track pool...")

    if choice == 1:
        # I set 40 as default to give us a good-sized pool to select from
        trackPool = generateTrackPool_tags(80, t, a)
        print(f"\nCreated a pool of {len(trackPool)} tracks!")

        # Let user customize playlist size
        try:
            playlist_size = int(input("\nHow many tracks in your playlist? (default: 15) ") or 15)
        except ValueError:
            print("Using default size of 15 tracks")
            playlist_size = 15

        # Generate the actual playlist using our tag-based algorithm
        playlist = generate_playlist_tags(trackPool, playlist_size)

        # Display the results nicely formatted
        title = f"Personalized Playlist based on {tempObjectT.name} and {tempObjectA.name}"
        display_playlist(playlist, title)

    if choice == 2:
        trackPool = generate_track_pool_popularity(t, a)
        print(f"\nCreated a pool of {len(trackPool)} tracks!")

        # Let user customize playlist size
        try:
            playlist_size = int(input("\nHow many tracks in your playlist? (default: 15) ") or 15)
        except ValueError:
            print("Using default size of 15 tracks")
            playlist_size = 15

        # Generate the actual playlist using our tag-based algorithm
        playlist = generate_playlist_popularity(trackPool, playlist_size)

        # Display the results nicely formatted
        title = f"Personalized Playlist based on {tempObjectT.name} and {tempObjectA.name}"
        display_playlist(playlist, title)


if __name__ == "__main__":
    main()
