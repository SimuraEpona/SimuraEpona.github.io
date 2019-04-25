---
title: 使用 Drafts 上传文件到 Github
tags: [Drafts, Github]
--- 

> 本文主要参考自 [Blogging from Drafts](https://roub.net/blahg/2019/01/14/blogging-from-drafts/index.html)

<!--more-->

## 首先

那么我们怎样才能使用 Drafts 来上传我们的博客文章呢？简单来讲，我们需要4个步骤：

1. 创建必要的[YAML header](https://jekyllrb.com/docs/front-matter/) 配置文件放在 markdown 最前面
2. 按照 Jekyll 的格式命名 markdown 文件
3. 把文件添加到项目的 `_posts` 文件夹下
4. 更新博客内容

第四步 Github 已经替我们做好了，当我们项目有更新的时候，Github 会自动进行更新。剩下的我们在 Drafts 里面搞定。

我们主要通过 Drafts 的 Action 来搞定


```
const credential = Credential.create("GitHub博客仓库", "需要用户名、仓库名和认证token");

credential.addTextField("username", "GitHub 用户名");
credential.addTextField('repo', '仓库名(无需前缀)');
credential.addPasswordField("key", "GitHub personal access token");

credential.authorize();

const githubKey = credential.getValue('key');
const githubUser = credential.getValue('username');
const repo = credential.getValue('repo');

const http = HTTP.create(); // create HTTP object
const base = 'https://api.github.com';


const txt = draft.content;
const postTime = new Date();

const datestr = `${postTime.getFullYear()}-${pad(postTime.getMonth() + 1)}-${pad(postTime.getDate())}`;
const timestr = `${pad(postTime.getHours())}:${pad(postTime.getMinutes())}`;
const slug = String((postTime.getHours() * 60 * 60) + (postTime.getMinutes() * 60) + postTime.getSeconds());

const fn = `${datestr}-${slug}.markdown`;

const yaml = {
    layout: 'post',
    title: '""'
};

let preamble = "---\n";

for (const f in yaml) {
    preamble += `${f}: ${yaml[f]}\n`;
}

preamble += "---\n\n";

const doc = `${preamble}${txt}`;

const options = {
    url: `https://api.github.com/repos/${githubUser}/${repo}/contents/_posts/${fn}`,
    method: 'PUT',
    data: {
        message: `Epona ${datestr}`,
        content: Base64.encode(doc)
    },
    headers: {
        'Authorization': `token ${githubKey}`,
			'Accept': 'application/vnd.github.v3+json'
    }
};

var response = http.request(options);

if (response.success) {
    // yay
} else {
    console.log(response.statusCode);
    console.log(response.error);
}

function pad(n) {
    let str = String(n);
    while (str.length < 2) {
        str = `0${str}`;
    }
    return str;
}

```

本质上我们要做的就是使用 GitHub 提供的 [API 接口](https://developer.github.com/v3/repos/contents/#create-a-file) 来上传我们的文件，我们需要在
[Personal Access Token](https://github.com/settings/tokens) 里申请一个 application，确保勾选上 repo 的所有权限。之后就可以用这个脚本开心的在手机或者 iPad 上随时写博客啦！

当然，可以优化的地方还有很多，剩下的地方就交给你们去自由发挥吧。

> 本周的一篇博客就水到这里吧😂，下周开始继续新的旅途。