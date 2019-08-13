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


you can turn this into a simple bash script that takes a video file as input :

```
usage : dvrupscaler -f /path/to/video/dvr/file.mov [-a /path/to/audio/file.mp3] [-v:verbose] [-s:shorten the final output to the shortest input file]
```

dvrupscaler script :

```
#!/bin/bash

################################################################################
# dvrupscaler - a DVR file upscaling script for social media video adaptation
# -> scales video to 1920x1080, converts to h264, nulls the audio track or uses 
# an mp3 audio file specified with the -a(udio) option. Should use the 
# -s(hortest) parameter # if your audio file is longer than the video.
# 
# Author : A.Arvinte (ledrone.club) - Aug 2019
# https://github.com/ledroneclub/tools
################################################################################

# Reset in case getopts has been used previously in the shell.
OPTIND=1 
# Initialize our variables
vfile=""
afile=""
verbose=0
shortest=0

logparams="-loglevel quiet -stats"
shortparams=""
audiosource="-f lavfi -i anullsrc"

while getopts "h?vf:a:s" opt; do
    case "$opt" in
    h|\?)
        echo "dvrupscaler -f video_file [–a audio_file] [-v:verbose] [-s:shorten to shortest input file]"
        exit 0
        ;;
    v)  verbose=1
        ;;
    a)  afile=$OPTARG
        audiosource="-i $afile"
        ;;
    f)  vfile=$OPTARG
        ;;
    s)  shortest=1
        ;;    
    esac
done

if [ $verbose -eq 1 ]; then
    logparams=""
fi

if [ $shortest -eq 1 ] || [ -z $afile ]; then
    shortparams="-shortest"
fi

shift $((OPTIND-1))
[ "${1:-}" = "--" ] && shift

file_pwd=$(dirname "${vfile}")
file_name_full=$(basename -- "$vfile")
file_ext="${file_name_full##*.}"
file_name="${file_name_full%.*}"
file_out=$file_pwd/${file_name}_upscaled.mp4

echo "Will encode $vfile to $file_out"


ffmpeg $logparams $audiosource -i $vfile \
    -pix_fmt yuv420p -c:v libx264 -b:v 8M \
    -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
    -c:a aac -map 0:a -map 1:v $shortparams $file_out
```

