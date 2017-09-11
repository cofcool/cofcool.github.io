---
layout: post
category : Snippets
title : 移除掘金网的“展开全文”按钮
tags : [snippets, js]
---
{% include JB/setup %}

```js
function removeShowFull() {
    var showFullBlocks = document.getElementsByClassName("show-full-block");
    if (showFullBlocks.length > 0) {
        // remove .show-full-block div
        var showFullBlock = showFullBlocks[0];
        showFullBlock.parentNode.removeChild(showFullBlock);

        // remove .show-full div
        var showFulls = document.getElementsByClassName("show-full");
        var showFull = showFulls[0];
        showFull.parentNode.removeChild(showFull);

        // show all
        var container = document.getElementsByClassName("post-content-container hidden");
        container[0].setAttribute("style", '"position": "absolute"');
    }
}

removeShowFull()
```
