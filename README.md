

### Description
发布生成的目录: `.deploy_git`, 移动时可不必要

Deploy to github pages or gitee pages
- Configuration file: `%HEXO_HOME%/_config.yml`
- Configuration property: deploy.repository
e.g.
```yml
deploy:
  type: git
  repository: git@gitee.com:thank037/thank037.git
  branch: master
```

### Deploy
- hexo clean
- hexo g
- hexo s (If you want to see it local)
- gulp 
- hexo d
