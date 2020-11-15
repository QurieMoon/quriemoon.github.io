---
title: "Google Colab 환경에서 Google Drive 연동하기"
date: 2019-04-29
category: Colab
tags:
- Colab
- Google_drive
---



 GPU가 없지만 딥러닝 모델을 학습시켜서 활용하고 싶거나 BERT와 같은 덩치가 큰 모델을 돌리고 싶을 때, GPU 장비와 같은 환경적인 조건이 갖추어져 있지 않아 시도조차 해보기 어려울 때가 종종 있다. 이때, 유용하게 사용할 수 있는 게 바로 Google Colab이다.

 이 글에서는 Google Colab 사용법 중에 하나로, Google Drive와의 연동을 통한 Google Drive 데이터를 불러오는 법을 살펴보려고 한다.

---------------------------------------------------

 이 글에서 등장하는 Google Drive 구조는 아래와 같다.

![gdrive_sample_directory]({{site.url}}/assets/images/post_images/gdrive_sample_directory.png)

 여기서 우리는 현재 작업 중인 위치를 'new_folder'로 설정한 후, 해당 폴더에 하위 항목('data' 폴더, sample.txt) 를 이용하려고 한다.

 다시  Colab으로 돌아와서, 구글 드라이브를 연동하기 전에 현재 작업 중인 위치와 접근 가능한파일을 살펴보면 아래와 같다.

```shell
!pwd # pwd: 현재 작업 중인 위치를 알려준다.
# /content
```

```shell
!ls # ls: 현재 작업 중인 디렉토리 내에 있는 디렉토리 및 파일 출력
# sample_data
```



아래 코드를 통해서 구글 드라이브를 연결해보자.

> 마운트?
>
> 컴퓨터 과학에서 저장 장치에 접근할 수 있는 경로를 디렉터리 구조에 편입시키는 작업을 말한다. (출처: 위키백과 - '마운트')

다음 코드를 실행하면, 결과창에 나오는 URL이 등장한다. 해당 URL을 클릭하여  authorization code를 복사해온 후, 입력창에 붙여넣자.

```python
# 구글 드라이브 연동

from os.path import join
from google.colab import drive
ROOT = "/content/gdrive"
drive.mount(ROOT)

# Go to this URL in a browser: https://accounts.google.com...

# Enter your authorization code:
# ··········
# Mounted at /content/gdrive
```

구글 드라이브와 연동을 마치고서 다시 현재 작업 중인 위치 및 세부 파일에 대한 정보를 확인해보자.

```shell
!pwd
# /content
```

```shell
!ls
# gdrive sample_data
```

구글 드라이브를 연동한 후에는 현재 작업 중인 디렉토리 안에 'gdrive' 라는 디렉토리가 보인다. 바로 이 'gdrive' 를 이용해서 구글 드라이브 내 파일에 접근할 수 있다.

 세부 경로를 설정하기 전에 `ls` 명령어를 이용하여 설정하려는 세부 경로가 맞는 경로인지 확인할 수 있다.

```shell
!ls /content/gdrive/'My Drive'/new_folder
# data sample.txt
```

 위 결과를 통해서 `/content/gdrive/'My Drive'/new_folder`라는 경로가 이 예제에서 설정하려는 경로가 맞다는 걸 확인했다.

> 팁: 디렉토리명에 ' ' (공백; white space) 가 포함된 경우, 하나의 디렉토리명이 공백을 기준으로 분리가 되어 에러가 발생할 수 있다. 이럴 경우, ' '으로 해당 디렉토리명을 묶어주면 된다.

위 경로를 이용하여 우리가 이용할 프로젝트 경로 (주소)를 만들자.

```python
PROJ = "My Drive/new_folder" #에러 발생 시, 아래 PROJ 할당문 실행
# PROJ = "\'My Drive\'/new_folder"
PROJECT_PATH = join(ROOT, PROJ) # 위에서 ROOT = "/content/gdrive"
```

> 팁:  위에서 언급했듯이, 공백이 포함된 디렉토리명이 포함된 경로 입력 시, ' '가 필요할 수도 있다. 이때, 문자열 안에 다시 문자열을 입력해야하는 경우, 이스케이스 문자(\\)를 쓰면 된다.

새로 만든 프로젝트 주소를 이용할 수 있도록 경로에 추가하자.

```python
import os
import sys
sys.path.insert(0, PROJECT_PATH)
```

우리가 이용하려는 `data` 디렉토리와 `sample.txt` 에 바로 접근할 수 있도록 new_folder 로 이동하자.

```shell
%cd gdrive/'My Drive'/new_folder
```

 `pwd` 명령어를 통해서 현재 작업 중인 위치가 `new_folder` 로 바뀌었음을 알 수 있다.

```shell
!pwd
# /content/gdrive/My Drive/new_folder
```

 이제 `pd.read_csv('sample.txt')` 와 같이 `new_folder` 에 있는 항목들에 바로 접근할 수 있다.