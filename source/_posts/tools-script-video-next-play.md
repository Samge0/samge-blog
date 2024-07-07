---
title: 提高追番效率：使用暴力猴脚本实现一键自动切换集数并跳过片头片尾
date: 2024-07-07 13:10:31
tags:
    - 追番
    - 脚本
    - 暴力猴
    - 油猴
    - 自动播放
    - 动漫
---

在某个动漫网站追番时，每次都要手动切换集数和调整进度以跳过片头片尾，这实在是太麻烦了。为了简化这个过程，我临时起意，让GPT帮忙编写了一个暴力猴（Tampermonkey）脚本，实现`"一键自动切换下一集并跳过片头"`的功能。

这个脚本让观看动漫的体验变得更加顺畅。现在，只需点一下按钮，就可以切换到下一集并自动跳过片头，极大地节省了重复操作的时间。

当然，也可以通过简单的设置实现全自动播放的功能，比如自动监听播放到片尾位置并切换到下一集。目前我观看的动漫片尾时间不固定，暂时没有配置全自动切换播放的功能。当前这个一键切换下一集的手动功能，已经解放了大量的重复时间，让观影体验更为愉悦。

以下是这个脚本的基本逻辑：
- **一键切换下一集**：无需手动选择下一集，只需点击一次按钮即可自动切换。
- **自动跳过片头**：在播放时自动跳过片头部分，直接进入正片。

一下是本次脚本的代码记录：
```js
// ==UserScript==
// @name         切换下一集按钮
// @namespace    Violentmonkey Scripts
// @match        https://www.mxdmp.com/play/*/1/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    let button = document.createElement('button');
    button.innerHTML = '切换下一集';
    button.style.position = 'fixed';
    button.style.left = '20px';
    button.style.top = '20px';
    button.style.zIndex = '10000';
    document.body.appendChild(button);

    button.addEventListener('click', function() {
        let currentUrl = window.location.href;
        let match = currentUrl.match(/https:\/\/www\.mxdmp\.com\/play\/(\d+)\/1\/(\d+)\//);
        if (!match) return;

        let dramaId = match[1];
        let currentEpisode = parseInt(match[2]);

        let nextEpisode = currentEpisode + 1;
        let jumpTime = 4 * 60; // 第三分钟的秒数
        let nextUrl = `https://www.mxdmp.com/play/${dramaId}/1/${nextEpisode}/?jumpTime=${jumpTime}`;

        // 跳转到下一集
        window.location.href = nextUrl;
    });

    // 当页面加载完毕时设置播放进度
    window.addEventListener('load', function() {
        let params = new URLSearchParams(window.location.search);
        let jumpTime = params.get('jumpTime');
        if (jumpTime) {
            let player = document.querySelector('.art-video'); // s根据实际情况获取视频元素
            if (player) {
                player.currentTime = parseInt(jumpTime);
                player.play();
            }
        }
    });
})();

```

如果想要播放到指定进度时自动播放下一集，可参考下面脚本（例如这里播放到`18:30`时自动播放下一集，完全自动化，解放双手）：
```js
// ==UserScript==
// @name         切换下一集按钮
// @namespace    Violentmonkey Scripts
// @match        https://www.mxdmp.com/play/*/1/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    let button = document.createElement('button');
    button.innerHTML = '切换下一集';
    button.style.position = 'fixed';
    button.style.left = '20px';
    button.style.top = '20px';
    button.style.zIndex = '10000';
    document.body.appendChild(button);

    button.addEventListener('click', function() {
        switchToNextEpisode();
    });

    function switchToNextEpisode() {
        let currentUrl = window.location.href;
        let match = currentUrl.match(/https:\/\/www\.mxdmp\.com\/play\/(\d+)\/1\/(\d+)\//);
        if (!match) return;

        let dramaId = match[1];
        let currentEpisode = parseInt(match[2]);

        let nextEpisode = currentEpisode + 1;
        let jumpTime = 4 * 60; // 第三分钟的秒数
        let nextUrl = `https://www.mxdmp.com/play/${dramaId}/1/${nextEpisode}/?jumpTime=${jumpTime}`;

        // 跳转到下一集
        window.location.href = nextUrl;
    }

    // 自动切换到下一集的逻辑
    function autoSwitchNextEpisode() {
        let player = document.querySelector('.art-video'); // 根据实际情况获取视频元素
        if (!player) return;

        player.addEventListener('timeupdate', function() {
            // 如果视频当前时间超过18:30秒，执行切换到下一集的操作
            if (player.currentTime >= 18 * 60 + 30) {
                switchToNextEpisode();
            }
        });
    }

    // 当页面加载完毕时设置播放进度
    window.addEventListener('load', function() {
        let params = new URLSearchParams(window.location.search);
        let jumpTime = params.get('jumpTime');
        if (jumpTime) {
            let player = document.querySelector('.art-video'); // 根据实际情况获取视频元素
            if (player) {
                player.currentTime = parseInt(jumpTime);
                player.play();
                autoSwitchNextEpisode(); // 启动自动切换下一集的监听
            }
        }
    });

})();

```