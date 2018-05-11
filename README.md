# GyuLog
GyuLog 의 원본 소스입니다 - https://gyukebox.github.io

## Installation

1. Install [hugo](https://gohugo.io)
```
$ brew install hugo
```

2. Clone this repo or pull from master

```
$ git clone https://github.com/gyukebox/gyulog-save
```

```
$ git remote add origin https://github.com/gyukebox/gyulog-save
$ git pull origin master
```

## Development

New post or draft:  
```
$ hugo new blog/<TITLE>.md
```

New static content:
```
$ hugo new static <CONTENT>
```

## Rendering
```
$ hugo server -D --watch
```

## Deployment

```
$ git remote add origin <REPOSITORY_NAME>
$ git pull origin master
$ git submodule add -b master https://github.com/gyukebox/gyukebox.github.io.git public 
$ ./deploy.sh "commit message"
```