# MTT - Move to tmp
## Override `rm` command to move files to /tmp instead permanently deleting them

> Disclaimer: the following code may cause loss of data. Please, backup your data before using it - although it should not be able to do more damage than running the native `rm` command.

### Intro

I was quite tired of losing files by `rm`ing them without thinking enough about the consequences of doing it.

I solved my issue by creating an alias to the `rm` command which essentially moves the files and folders I am about to remove to the system temporary folder, i.e. `/tmp`. This folder is emptied by the OS upon reboot.

In case I want to use the native `rm` command, I call `/bin/rm` instead of `rm` only. 

### Implementation

To implement my solution, append the following code to `~/.bash_profile`.

```bash
alias rm="move_to_tmp"
move_to_tmp() {
  # Folder where the "deleted" files will be moved
  TRASH="/tmp"
  
  RECURSIVE=false
  # Parse options
  if [[ $1 == "-"* ]]; then
  OPTION_LIST=(`echo $1 | sed 's/^-//' | sed -e's/\(.\)/\1 /g'`)
    for CHAR in $OPTION_LIST; do
      case "$CHAR" in
        r)
          RECURSIVE=true
          ;;
        
        *)
        echo "Invalid option: $CHAR";
        return 1
      esac
    done
  fi

  # in case a list of files is passed
  for INPUT in $@; do
    # skip the parameter input
    if [[ $INPUT == "-"* ]]; then
      if ! [ "$INPUT" = "$1" ]; then
        "rm: $INPUT must be specified as first parameter" >&2
        echo >&2
        return 1
      fi
      continue
    fi

    # check file presence
    # echo "Checking $INPUT"
    if [ ! -f "$INPUT" ] && [ ! -d "$INPUT" ]; then
      echo "move_to_trash: $INPUT: No such file or directory" >&2
      echo
      return 1;
    fi

    FILE_NAME=`basename $INPUT`
    INDEXED_FILE_NAME="$FILE_NAME"
    # check that the same file is in the "$TRASH"/ folder already
    if [ -f "$TRASH"/"$FILE_NAME" ] || [ -d "$TRASH"/"$FILE_NAME" ]; then
      # create a file name with variable index and increment the index until
      #+a file with the same name and index does not exist
      INDEX=2
      INDEXED_FILE_NAME="${FILE_NAME}-$INDEX"
      # echo "New file name: $INDEXED_FILE_NAME"
      while [ -f "$TRASH"/"$INDEXED_FILE_NAME" ] || [ -d "$TRASH"/"$INDEXED_FILE_NAME" ]; do
        INDEX=$((INDEX + 1))
        INDEXED_FILE_NAME="${FILE_NAME}-$INDEX"
        echo "New file name: $INDEXED_FILE_NAME"
      done
    fi
    # Check that -r was specified if the current INPUT is a folder
    if [ $RECURSIVE = false ] && [ -d $INPUT ]; then
      echo "rm: $INPUT: is a directory" >&2
      echo "specify '-r' if you want to remove it" >&2
      echo >&2
      return 1
    fi
    mv -v "$INPUT" "$TRASH"/"${INDEXED_FILE_NAME}"
  done
}
```

Open a new terminal window (it won't work on an already open terminal) and run the following commands for testing

```
$ mkdir tmp_folder
$ touch tmp_file
$ rm -r tmp_folder tmp_file
$ ls /tmp
```
Run the same commands again and you'll see that you'll have stored in `/tmp/` all the files you have deleted, conveniently renamed not to overwrite duplicates.

### How it works

The first line in the above code overrides the `rm` command with the function that follows. As expressed by the function name, the files will be moved to `/tmp/` folder and will stay there until the next reboot. If `/tmp` already contains files or folders with the same name as the one you are `rm`ing, those are renamed by using an integer as unique identifier which will be appended to or modified in the file or folder name.\

Optionally, you can modify the script to

- move files and folders to some other location (e.g. `~/.Trash` on MacOS) by changing the `TRASH` parameter
- call the alias differently by modifying the alias name from `rm` to `something_else_you_like`

> NOTE: Arguments to `rm` are parsed as options by looking for dashes at the beginning of the parameter. Hence, although it's very unusual, if you have a file or folder whose name starts with `-` (dash), you cannot remove it by calling `rm [-r] -file_or_folder_name` because `file_or_folder_name` would be interpreted as an option. Instead, you can specify the relative or full path of what you want to remove, e.g. `rm [-r] ./-file_or_folder_name`.
