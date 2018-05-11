+++
title = "Git 입문자를 위한 git 사용법"
categories = ["git"]
tags = ["git"]
date = "2017-08-30 01:24:37"
+++

![](/img/git.png)

개발할 때 코딩 실력이나만큼 중요한 것은 바로 버전 관리죠. 현재 거의 필수적으로 쓰이는 버전 관리 툴인 git 에 관하여 알아보는 포스트입니다. (git 을 아예 모르는 사람들을 대상으로 포스팅했습니다.)

<!-- more -->

해당 포스트에서는 터미널 명령어를 사용합니다. 터미널이 아직까지는 어려우신 분들은 `SourceTree` 를 활용하셔도 좋습니다(`SourceTree`에 관해서는 아래에 자세하게 명시하겠습니다).

## 버전 관리 툴의 필요성

학교에서 팀 프로젝트를 해봤다면 이런 경험이 한 번쯤은 있었을 것이다.

```
철수 : (압축파일을 푼다) 영희야, 너가 보내준 파일 받았는데 여기 OOOO한 부분에서 문제가 있어.
      그리고 그때 말한 그 부분 내가 만들었어. 파일 보냈어! (코드를 압축해서 카카오톡으로 보낸다)
영희 : (압축파일을 푼다) 그런데 철수야, 그렇게 만들면 내가 만든 부분이랑 자료형이 달라서 안 맞아.
      이러이렇게 만들어서 다시 보내줘. 내가 만든 부분을 지금 수정하기 힘들어.
철수 : (한숨을 쉬며) 그래 알았어... (새로 만든 코드를 압축해서 카카오톡으로 보낸다)
      여기 다 만들었다.
영희 : 헉 잠시만 철수야, 앞에 만든 부분이 문제가 생겨서 수정해야 할 것 같거든?
      (압축파일을 풀고 수정을 시도하지만, 역시 안 맞는 부분이 있다)
      그런데 이제는 ㅁㅁㅁㅁ한 부분에서 안 맞아. 다시 수정해야 되겠는데?
철수, 영희 : 하...
```

보기만 해도 답답한 ~~암걸리는~~ 상황이다. 이럴 때, 서로의 코드에서 다른 부분을 알아서 잡아주는 도구가 있었으면 얼마나 편했을까?  
또한, 카카오톡으로 매번 압축해서 파일을 보내고 압축을 풀고... 이런 불편한 작업 없이 소스코드를 한 곳에 모아서 관리할 수 있는 도구가 있었으면 얼마나 좋았을까?

이런 상황에서 사용하도록 만들어진 도구가 `git` 이다! `git`을 이용하면 위의 상황에서 훨씬 편리하게 작업중인 소스코드를 관리하는 것이 가능하다. 이제 `git`에 관하여 자세하게 알아보자!

## Git 설치하기

가장 먼저 할 일은, 역시나 설치이다. 설치 방법은 다음과 같다.

#### Linux

리눅스 배포판에 따라 다음 두 명령어 중 하나를 입력한다. (하나가 안되면 다른 거다)

```
$ sudo yum install git
```

```
$ sudo apt-get install git
```

#### macOS

`Homebrew`가 설치되어 있다면, 다음 명령어만 입력하면 된다.

```
$ brew install git
```

(Homebrew 는 맥용 패키지 관리 도구입니다. 설치 방법 및 자세한 내용은 https://brew.sh/index_ko.html 를 참고하세요.)

#### Windows

안타깝게도, 윈도우에서는 커맨드라인 하나만으로 git 을 설치할 수 없기에, 공식 사이트(https://git-scm.com) 를 통해 설치하거나, `SourceTree`나 `Github Desktop` 같은 GUI 도구를 이용해야 한다. Command line tool 은 https://git-for-windows.github.io 에서 다운로드 받을 수 있다.

(사실 초보자들한테는 GUI 툴이 더 시작하기는 좋습니다.)  
(윈도우 10 부터는 cmd 에서 bash 명령어를 이용하는 것이 가능해졌습니다. 하지만, 많이 부족하여 아직 사용하기는 시기상조다 라는 의견이 많습니다.)

운영체제에 맞게 git 을 설치하였으면, 사용할 준비가 되었다!

## Git 시작하기

이제 git 을 본격적으로 사용해 보자!

### 저장소 생성하기

철수는 저번 프로젝트를 교훈삼아, 이번 프로젝트에서는 꼭 버전 관리 도구를 사용하기로 마음먹었고, git 을 사용하기로 하였다.  
우선, 그는 자신이 작업중인 프로젝트 폴더로 들어가서, 다음 명령어로 git 을 시작했다.

```
$ cd /path/to/project
$ git init
```

`git init` 명령어를 사용하면 현재 작업중인 프로젝트 폴더에 `.git` 이라는 숨김 파일이 만들어지고,
"git 저장소"가 생성된다. git 은 `.git` 파일이 위치하는 디렉토리 및 하위 디렉토리에 위치해 있는 파일들을 관리한다.  
만약 각자 다른 프로젝트를, 따로 관리하고 싶다면, 각각의 프로젝트 폴더에서 각자 `git init` 을 사용해 주어야 한다.

### .gitignore 파일

하지만, 모든 파일을 버전 관리에 포함하고 싶지는 않다. 가령 소스 파일을 빌드해서 만들어진 실행파일이나, 이미지 파일이나, 아니면 IDE 에서 자동으로 생성되지만 필요가 없는 폴더의 파일들(예: JetBrains IDE 의 `.idea`) 같은 경우는, 딱히 버전 관리를 해줄 필요가 없다.  
Git 은 이렇게 사용자가 버전 관리를 하고 싶지 않은 파일들을 무시할 수 있는 기능이 있다. Git 이 무시하도록 하는 파일들을 명시하는 파일이 `.gitignore` 파일이다.

다시 철수로 돌아가보자. 철수는 자신이 작업하고 있는 프로젝트의 파일 중, 이미지 파일을 포함한 몇몇 파일은 버전 관리의 필요성을 느끼지 않아서, `.gitignore` 파일에 이를 추가했다.

```
.gitignore
*.jpg
*.png
.idea/*
.DS_Store
```

(해당 예시는 그냥 예시일 뿐입니다. 꼭 이렇게 작성하라는 뜻이 아니라, 여러분들의 필요에 맞게 `.gitignore` 파일을 작성하시면 됩니다.)

예시를 자세히 보면 `.gitignore` 파일에서 무시 파일을 명시하는 방법은 아래와 같이 세 가지가 있음을 알 수 있는데,

* 확장자를 명시하여 해당 확장자의 파일을 모두 무시
* 디렉토리를 명시하여 해당 디렉토리 아래의 파일을 모두 무시 (이 경우 subdirectory 도 포함)
* 파일 이름을 정확히 명시하여 해당 파일을 무시

뭐 상황에 맞게 세 가지 방법 중 하나를 사용하면 된다.

## 파일의 상태

Git 에서는 현재 작업중인 프로젝트 파일의 상태를 다음과 같이 세 가지로 분류한다.

* 작업 중인 상태
* 준비 상태
* 완료 상태

그리고, git 저장소에는 (눈에는 보이지 않지만) 파일들의 상태를 구분하기 위하여 세 종류의 영역이 있는데,

* `working directory` : 현재 작업 중인 실제 파일들
* `index` : 작업이 완료되어 확정을 준비하는 파일들 (이 영역에 파일들을 추가하는 행동을 `stage` 라고 한다)
* `HEAD` : 최종 확정본들 (이 영역에 파일을 추가하는 행동을 `commit` 이라고 한다)

~~왜 HEAD 만 대문자일까~~  
위와 같다. 그렇다면, git 에서 소스코드를 관리하는 과정은 다음과 같다는 것을 알 수 있게 된다.

1. 소스 코드 작업을 완료한다.
2. 작업이 완료된 코드를 `index`에 `stage` 한다 (또는 스테이지에 올린다).
3. 필요한 파일들을 모두 `stage` 했으면 묶어서 `HEAD`에 `commit`(커밋) 한다.

~~어때요 참 쉽죠?~~  
예를 들어, 철수는 웹사이트를 하나 제작하기 위해서 맨 처음 화면인 `index.html` 을 처음 만들었다고 하자. 그러면 철수는 이 "`index.html` 제작" 을 하나의 버전(단계) 로 생각하여, git 에서 `stage` 후 `commit` 으로 관리할 수 있게 된 것이다.

## 상태 확인과 staging

철수는 소스 코드 작업을 마치고 현재 git 저장소의 상태를 알아보기 위하여 다음 명령어를 사용했다.  
`git status`  
이 명령어를 입력하니 다음과 같이 현재 파일들의 상태가 한 눈에 보였다.

```
$ git status

On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	index.html

nothing added to commit but untracked files present (use "git add" to track)
```

`git status` 명령어를 사용하면, 현재 어떤 파일이 추가되고 삭제되었는지, 어떤 파일이 아직 추가되지 않았는지 볼 수 있다. 심지어 "추가하려면 `git add` 를 쓰세요" 라고 친절하게 알려준다! 그렇다, 파일을 `index`에 `stage` 하고 싶으면 `git add`를 사용하면 된다.

철수는 작업이 완료된 파일을 스테이지에 올리기 위하여 다음과 같이 진행했고, 상황을 한번 더 확인했다.

```
$ git add index.html
$ git status

On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   index.html
```

`git add <파일이름>` 을 사용해서 해당 파일을 스테이지에 올릴 수 있다.  
`git status` 를 사용하면, 스테이지에 올라간 파일들은 "change to be committed: " 라는 문구 아래에 놓이고, 아직 스테이지에 올라가지 않은 파일들은 "untracked files: " 라는 문구 아래에 놓이게 된다.

하지만, 일일이 모든 파일을 추가하자니 좀 귀찮은 감이 있다. 위와 같이 하나의 파일만을 추가하는 명령 말고도, 다음과 같은 표현들을 사용하면 한번에 여러 개의 파일들을 동시에 스테이지에 올릴 수가 있다.

* `git add *.foo` : foo 확장자를 가진 모든 파일들을 스테이지에 올린다.
* `git add dir/*` : dir 디렉토리 안의 모든 파일들을 스테이지에 올린다.
* `git add --all` : 아직 스테이지에 올라가지 않은 모든 파일들을 스테이지에 올린다.

하지만, 동일 확장자나 동일 디렉토리 안에 현재 버전에 포함시키고 싶지 않은 파일이 들어 있을 수도 있으므로 신중하게 써야 한다.

## Commit 과 commit message

철수는 필요한 모든 파일들을 스테이지에 올렸다. 스테이지에 올라간 파일들을 묶어서 커밋하기 위하여 다음과 같이 명령어를 사용하였다.

```
$ git commit -m "Initial commit : added index.html"
[master (root-commit) 772f2e6] Initial commit : added index.html
 1 file changed, 8 insertions(+)
 create mode 100644 index.html

$ git status
On branch master
nothing to commit, working tree clean
```

git 에 내용을 커밋할 때는 위와 같이 `git commit -m "commit message"` 라는 명령어를 사용한다. 그러면 친절하게 커밋의 내용을 보여준다. 또한 커밋 이후에 `git status` 로 상태를 확인하면 다음과 같이 보이게 된다. (물론 아직 커밋을 안 한 파일이 있으면 다르게 나온다.)

앞의 과정들과 약간 다른 점은, 커밋을 할 때에는 커밋 메시지(commit message) 를 작성한다는 점인데, 현재 개발자가 무슨 작업을 진행했는지 알 수 있도록 하기 위해서다. 따라서, 커밋 메시지는 **간결하지만 자세하게** 작성해야 한다! (~~이것은 마치 따뜻한 냉커피..~~)  
길지 않으면서, 현재 작업을 통하여 예전과 변경된 점이 명확하게 드러나도록 작성하면 좋은 커밋 메시지가 될 수 있다.

## 원격 저장소

철수는 이제 작업한 내용을 영희한테 보여줄 준비가 되었다. 하지만 또 파일을 압축해서 카카오톡으로 보낸다는 건 상상도 하기 싫은 일. 따라서, 현재 저장소와 연동할 수 있는 **원격 저장소** 를 두기로 하였다. 그래서, 다음과 같이 했다.

우선 `GitHub` 에서 레포지토리를 하나 만들었다. 그 다음, 아래의 명령어로 깃허브 원격 저장소랑 현재 저장소를 연동시켰다.

```
$ git remote add origin https://github.com/gyukebox/gittutorial.git
```

(GitHub 저장소는 그냥 만들면 됩니다. 깃허브 가서 Create Repository 누르면 친절하게 나옵니다.)  
`git remote add [단축이름] [url]` 명령어를 사용하여 url 의 원격 저장소를 단축 이름을 사용하여 현재 git 저장소랑 연동시켰다(github 말고도 다른 원격 저장소들도 있지만, 거의 다 github 를 사용하므로 예시도 그냥 github 로 들겠습니다).

현재 저장소에 연동된 원격 저장소들을 보려면 다음과 같은 명령어를 사용하면 된다.

```
$ git remote -v
origin	https://github.com/gyukebox/gittutorial.git (fetch)
origin	https://github.com/gyukebox/gittutorial.git (push)
```

여기서 `-v` 옵션은 verbose 의 의미로, "자세히" 라는 뜻이다. 저 옵션을 붙이지 않으면 그냥 단축 이름만 나오지만, 옵션을 붙이면 원격 저장소 url 과 같이 나온다.

### 원격 저장소에 올리기(push)

이제 원격 저장소도 만들었으니, 원격 저장소에 현재 작업한 내용들을 올리는 일만 남았다. 철수는 다음과 같이 현재 작업 내용을 올렸다.

```
$ git push origin master
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 314 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/gyukebox/gittutorial.git
 * [new branch]      master -> master
```

`git push [원격 단축이름] [브랜치]` 명령어를 사용하면, 해당 단축이름을 가지는 원격 저장소로 코드가 올라가게 된다. 여기서 `master`는 **브랜치** 인데, 2 명 이상이 협업할 때 아주 중요한 개념이 된다(`master`는 기본 브랜치 이름입니다. 브랜치에 관해서는 아직 자세히 모르셔도 됩니다. 추후 포스팅으로 찾아뵙겠습니다).

이제 철수는 원격 저장소를 통하여 소스 코드를 올리므로 더 이상 영희에게 파일을 압축하여 카카오톡으로 보내지 않아도 된다! 영희가 해당 저장소 링크로 들어가서 코드를 확인하면 그만이니깐.

## 요약

`git` 에 관한 아주 기본적인 개념들을 알아봤는데, 간단하게 정리하면 다음과 같이 요약할 수 있다.

* `git init` : 새로운 git 저장소를 만든다.
* `git status` : 현재 저장소의 상황을 본다.
* `git add <file>` : 파일을 스테이지에 올린다.
* `git commit -m "commit message"` : 메시지와 함께 커밋한다.
* `git remote add [name] [url]` : 원격 저장소를 추가한다.
* `git push [remote] [branch]` : 원격 저장소에 올린다.

## 앞으로 더 알아볼 내용

(추후 포스팅으로 해당 개념들에 대하여 설명하겠습니다.)

* 원격 저장소에서 가져오기 : `pull` 과 `clone`
* 원격 저장소의 설명을 담당하는 `README.md`
* 브랜치(branch)
* `pull request` 와 `merge`
* `rebase`

## 참고한 글들

* https://rogerdudler.github.io/git-guide/index.ko.html
* http://blog.weirdx.io/post/45529
