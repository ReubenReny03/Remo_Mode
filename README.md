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
