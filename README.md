![image](https://github.com/user-attachments/assets/866f7d2e-e02e-46c6-9753-45e78159a1f1)

```
# Tab completion for `remo` command
_remo_completions() {
  local cur prev alias_list brain_list
  COMPREPLY=()
  cur="${COMP_WORDS[COMP_CWORD]}"
  prev="${COMP_WORDS[COMP_CWORD-1]}"

  # Main commands
  local commands="add remove brain --help"

  # Completion at position 1: `remo <TAB>`
  if [[ $COMP_CWORD -eq 1 ]]; then
    COMPREPLY=( $(compgen -W "$commands" -- "$cur") )

  # Completion for `remo remove <alias>`
  elif [[ $COMP_CWORD -eq 2 && $prev == remove ]]; then
    alias_list=$(grep -oE '^alias +[^=]+' "$HOME/Documents/REMO_MODE/aliases.sh" | awk '{print $2}')
    COMPREPLY=( $(compgen -W "$alias_list" -- "$cur") )

  # Completion for `remo brain <title>` (list existing brain files)
  elif [[ $COMP_CWORD -eq 2 && $prev == brain ]]; then
    brain_list=$(ls "$HOME/Documents/REMO_BRAIN" 2>/dev/null)
    COMPREPLY=( $(compgen -W "$brain_list" -- "$cur") )
  fi
}
complete -F _remo_completions remo
```
```
remo@fedora:~/Documents/CLI_MUSIC$ cat main.py 
import json
import subprocess
from ytmusicapi import YTMusic
from rich import print
from rich.prompt import Prompt, IntPrompt

ytmusic = YTMusic()


def search_music(query):
    results = ytmusic.search(query, filter="songs")
    top_results = results[:5]
    music_list = []

    print(f"\n[bold green]Top Results for:[/bold green] [blue]{query}[/blue]\n")
    for index, song in enumerate(top_results):
        title = song['title']
        artist = song['artists'][0]['name']
        video_id = song.get('videoId')
        url = f"https://www.youtube.com/watch?v={video_id}"
        print(f"[{index+1}] {title} by {artist}\n    [cyan]{url}[/cyan]")
        music_list.append((title, artist, url))

    return music_list


def save_song(song):
    title, artist, url = song
    data = {"title": title, "artist": artist, "url": url}

    try:
        with open("favorites.json", "r") as f:
            favs = json.load(f)
    except FileNotFoundError:
        favs = []

    favs.append(data)
    with open("favorites.json", "w") as f:
        json.dump(favs, f, indent=2)

    print(f"[green]✔ Saved:[/green] {title} by {artist}")


def play_with_mpv(url):
    try:
        cmd = f"yt-dlp -f ba --get-url {url}"
        audio_url = subprocess.check_output(cmd, shell=True, text=True).strip()
        print(f"[blue]🎧 Playing with mpv:[/blue] {url}")
        subprocess.run(["mpv", audio_url])
    except subprocess.CalledProcessError:
        print("[red]❌ Failed to fetch audio stream or play it.[/red]")


def view_saved():
    try:
        with open("favorites.json", "r") as f:
            favs = json.load(f)

        print("\n[bold]Your Saved Tracks:[/bold]")
        for i, song in enumerate(favs, 1):
            print(f"[{i}] {song['title']} by {song['artist']} - [cyan]{song['url']}[/cyan]")

        play = IntPrompt.ask("Enter number to play (0 to skip)", default=0)
        if 1 <= play <= len(favs):
            play_with_mpv(favs[play - 1]["url"])

    except FileNotFoundError:
        print("[yellow]No saved songs yet.[/yellow]")


def main():
    while True:
        print("\n[bold]🎵 YouTube Music CLI[/bold]")
        print("[1] Get Recommendations")
        print("[2] View Saved Songs")
        print("[3] Exit")

        choice = Prompt.ask("Choose an option", choices=["1", "2", "3"])

        if choice == "1":
            mood = Prompt.ask("Enter a mood, genre or artist")
            results = search_music(mood)

            to_save = IntPrompt.ask("Enter song number to save (0 to skip)", default=0)
            if 1 <= to_save <= len(results):
                save_song(results[to_save - 1])

            to_play = IntPrompt.ask("Enter song number to play (0 to skip)", default=0)
            if 1 <= to_play <= len(results):
                play_with_mpv(results[to_play - 1][2])

        elif choice == "2":
            view_saved()

        elif choice == "3":
            print("[bold red]Goodbye![/bold red]")
            break


if __name__ == "__main__":
    main()
```
