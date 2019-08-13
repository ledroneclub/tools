# various useful stuff for FPV


### DVR video Upscaling

Because you may want to remove noisy audio, rescale to 1080p your dvr video in order to share it on YouTube or other. This is a "simple" ffmpeg oneliner to achieve that.

```
ffmpeg -f lavfi -i anullsrc -i DVR_VIDEO _FILE.mov \
	-pix_fmt yuv420p -shortest -c:v libx264 -b:v 8M \
	-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
	-c:a aac -map 0:a -map 1:v \
	VIDEO_FILE_upscaled.mp4
```

the output will be an h264 encoded mp4 and your original 640x480 video will be upscaled to 1920x1080 while respecting the proportions.


you can turn this into a simple bash script that takes a fideo file as input :

```
usage : dvrupscaler -f /path/to/video/dvr/file.mov [-v]
```

dvrupscaler script :

```
#!/bin/bash

# dvrupscaler - a DVR file upscaling script for social media video adaptation
# scales video to 1920x1080, converts to h264, nulls the audio track
# Author : A.Arvinte (ledrone.club) - Aug 2019
# https://github.com/ledroneclub/tools

OPTIND=1 # Reset in case getopts has been used previously in the shell.

# Initialize our variables
file=""
verbose=0
logparams="-loglevel quiet -stats"

while getopts "h?vf:" opt; do
    case "$opt" in
    h|\?)
        echo "dvrupscaler -f video_file [-v]"
        exit 0
        ;;
    v)  verbose=1
        ;;
    f)  file=$OPTARG
        ;;
    esac
done

if [ $verbose -eq 1 ]; then
    logparams=""
fi

shift $((OPTIND-1))
[ "${1:-}" = "--" ] && shift

file_pwd=$(dirname "${file}")
file_name_full=$(basename -- "$file")
file_ext="${file_name_full##*.}"
file_name="${file_name_full%.*}"
file_out=$file_pwd/${file_name}_upscaled.mp4

echo "Will encode $file to $file_out"

ffmpeg $logparams -f lavfi -i anullsrc -i $file \
    -pix_fmt yuv420p -shortest -c:v libx264 -b:v 8M \
    -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
    -c:a aac -map 0:a -map 1:v $file_out
```

