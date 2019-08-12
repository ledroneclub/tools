# various useful stuff for FPV


### DVR video Upscaling

Because you may want to remove noisy audio, rescale to 1080p your dvr video in order to share it on YouTube or other. This is a "simple" ffmpeg oneliner to achieve that.

```
ffmpeg -f lavfi -i anullsrc -i DVR_VIDEO _FILE.mov \
	-pix_fmt yuv420p -shortest -c:v libx264 \
	-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
	-c:a aac -map 0:a -map 1:v \
	VIDEO_FILE_upscaled.mp4
```

the output will be an h264 encoded mp4 and your original 640x480 video will be upscaled to 1920x1080 while respecting the proportions :)


you can turn this into a simple bash script that takes a fideo file as input :

```
usage : dvrupscaler -f /path/to/video/dvr/file.mov
```

dvrupscaler script :

```
#!/bin/bash

# A POSIX variable
OPTIND=1         # Reset in case getopts has been used previously in the shell.

# Initialize our own variables:
file=""
verbose=0

while getopts "h?vf:" opt; do
    case "$opt" in
    h|\?)
        echo "dvrupscale -f video_file"
        exit 0
        ;;
    v)  verbose=1
        ;;
    f)  file=$OPTARG
        ;;
    esac
done

shift $((OPTIND-1))

[ "${1:-}" = "--" ] && shift

# echo "verbose=$verbose, input_file='$input_file', Leftovers: $@"

file_pwd=$(dirname "${file}")
file_name_full=$(basename -- "$file")
file_ext="${file_name_full##*.}"
file_name="${file_name_full%.*}"
file_out=$file_pwd/${file_name}_upscaled.mp4


#echo $file_pwd $file_name_full $file_name $file_ext
#echo $file_pwd/${file_name}_upscaled.mp4
#exit 0

echo "Will encode $file to $file_out"

ffmpeg -loglevel quiet -stats -f lavfi -i anullsrc -i $file \
    -pix_fmt yuv420p -shortest -c:v libx264 -b:v 8M \
    -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
    -c:a aac -map 0:a -map 1:v $file_out
```

