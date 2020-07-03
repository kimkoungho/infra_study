# git config 수정하기 

git config 는 global 설정과 repos 설정이 있음  

## remote 수정하기
git remote 는 원격 git repository url 인데 프로젝트의 repo 위치가 변경되는 경우 수정할 수 있음  

```shell script
# git remote 확인 
$ git remote -v 
origin  https://github.com/kimkoungho/infra_study.git (fetch)
origin  https://github.com/kimkoungho/infra_study.git (push)

#   
$ git remote set-url origin https://github.com/user/repo2.git
# Change the 'origin' remote's URL
# 

$ git remote -v  
origin  https://github.com/user/repo2.git (fetch)
origin  https://github.com/user/repo2.git (push)
```

참고 
[misone 깃 블로그](http://minsone.github.io/git/github-managing-remotes-changing-a-remotes-url)

## git clone 프로젝트 이름 지정 
```shell script
# git clone <Repo> <DestinationDirectory>
$ git clone https://github.com/user/repos.git my-custom 
```

[stackoverlfow](https://stackoverflow.com/questions/8570636/change-name-of-folder-when-cloning-from-github)
