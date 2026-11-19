# hugo-blog

Hugo + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 개인 블로그.

## 시작하기
```bash
brew install hugo
git clone --recurse-submodules <repo-url>   # 이미 clone했다면: git submodule update --init
hugo server -D          # http://localhost:1313
hugo new content posts/my-post.md
hugo --gc --minify      # public/ 빌드
```

## 테마 업데이트
```bash
git submodule update --remote --merge
```
