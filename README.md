# various useful stuff for FPV


### DVR video Upscaling

Because you may want to remove noisy audio, rescale to 1080p your dvr video in order to share it on YouTube or other. This is a "simple" ffmpeg oneliner to achieve that.

'''
ffmpeg -f lavfi -i anullsrc -i DVR_VIDEO _FILE.mov \
	-pix_fmt yuv420p -shortest -c:v libx264 \
	-vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1" \
	-c:a aac -map 0:a -map 1:v \
	VIDEO_FILE_upscaled.mp4
'''

the output will be an h264 encoded mp4 and your original 640x480 video will be upscaled to 1920x1080 while respecting the proportions :)
