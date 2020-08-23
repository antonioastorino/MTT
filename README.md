# MTT - Move to tmp
## Override `rm` command to move files to /tmp instead permanently deleting them

> Disclaimer: the following code may cause loss of data. Please, backup your data before using it - although it should not be able to do more damage than running the `rm` command.

I was quite tired of losing files by removing them withouth thinking enough about the consequences of doing it.

This is how I solved my issue, until the next reboot.

In `~/.bash_profile`, add the following

```bash
alias rm="move_to_tmp"
move_to_tmp() {
	RECURSIVE=false
	if [[ $1 == "-"* ]]; then
	PARAM_LIST=(`echo $1 | sed 's/^-//' | sed -e's/\(.\)/\1 /g'`)
		for CHAR in $PARAM_LIST; do
			case "$CHAR" in
				r)
					RECURSIVE=true
					;;
				
				*)
				echo "Invalid parameter: $CHAR";
				return 1
			esac
		done
	fi

	# in case a list of files is passed
	for INPUT in $@; do
		# skip the parameter input
		if [[ $INPUT == "-"* ]]; then
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
		# check that the same file is in the /tmp/ folder already
		if [ -f /tmp/"$FILE_NAME" ] || [ -d /tmp/"$FILE_NAME" ]; then
			# create a file name with variable index and increment the index until
			#+a file with the same name and index does not exist
			INDEX=1
			INDEXED_FILE_NAME="${FILE_NAME}-$INDEX"
			echo "New file name: $INDEXED_FILE_NAME"
			while [ -f /tmp/"$INDEXED_FILE_NAME" ] || [ -d /tmp/"$INDEXED_FILE_NAME" ]; do
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
		mv -v "$INPUT" /tmp/"${INDEXED_FILE_NAME}"
	done
```

The first line overrides the `rm` command with the function that follows. As expressed by the function name, the files will be moved to `/tmp/` folder and will stay there until the next reboot. Alternatively, you can modify the script to move files and folders to some other location (e.g. `~/.Trash` on MacOS). Up to you.
