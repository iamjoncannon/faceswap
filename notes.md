https://github.com/deepfakes/faceswap

# sourcing photos

see scraper for scraping webpages

convert videos to photos

```bash
ffmpeg -i ./jlo.mp4 -vf fps=5 ./photos/JLo/video-frame-%d.png

ffmpeg -i ./jlo.mp4 -vf fps=5 ./photos/JLo/video-frame-%d.png

ffmpeg -i ./bermi_video.mp4 -vf fps=100 ./photos/src/video-frame-%d.png
```

two separate folders, one for source video and another for JLo

autocrop JLo to remove exterior

```bash
autocrop -o output -r rejects -i . --facePercent 100
```

photos/JLo # these are cropped
photos/src # these aren't cropped- they're the source

other libraries
https://github.com/leblancfg/autocrop
https://gitlab.com/wavexx/facedetect
https://github.com/yxlijun/S3FD.pytorch

# full procedure on remote machine

EC2 AMI should be p series- 
	p2xlarge $0.90 per hour
	p2x8large $7.20 per hour

https://aws.amazon.com/marketplace/pp/B077GCH38C

## install

the EC2 comes with most of the dependencies installed

```bash
pip install -r requirements.txt
python setup.py
```
no docker, yes CUDA
there are a few small install issues to iron out

## extract

NB: Parallel processing disabled. You may get faster extraction speeds by enabling it with the -mp switch

python faceswap.py extract -i ./vids/JLo/output -o ./data/JLo -mp && 
python faceswap.py extract -i ./vids/src -o ./data/src -mp && 
python faceswap.py extract -i ./setOne -o ./data/JLo -mp

## train

python faceswap.py train -A ./data/src -B ./data/JLo -m ./models -t original

## convert

python faceswap.py convert -i ./vids/src -o ./data/output -m ./models

python faceswap.py convert -i ./vids/src -o ./data/output -m ./models -ref bermi_video.mp4

(migrate back to local machine)

aws s3 sync data/output/ s3://jaylow/OUTPUT

aws s3 sync s3://jaylow/OUTPUT SWAPPEDVID/
264MB

(call to convert back into video)

ffmpeg -i video-frame-%0d.png -c:v libx264 -vf "fps=100,format=yuv420p" out.mp4

ffmpeg -i video-frame-%0d.png -c:v libx264 -vf "fps=30,format=yuv420p" out.mp4

( fps of actual video ~30 format: H.264 640 x 352 )


ffmpeg -i out.mp4 -vcodec libx264 -crf 20 JayLow.mp4

# migrate to bigger EC2

## relay model through S3 to other instance

on old machine:
aws s3 sync models/ s3://iamjoncannonscrapedata/THEMODEL

on new machine:
aws s3 sync s3://iamjoncannonscrapedata/THEMODEL models/
aws s3 sync s3://iamjoncannonscrapedata/jaylow vid/
