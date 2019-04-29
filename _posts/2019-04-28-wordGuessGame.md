---
layout:     post
title:     WordGuessGame
subtitle:   wordGuessGame
date:       2019-04-28
author:     KT
header-img: img/post-bg-wdguess-web.jpg
catalog: true
tags:
    - Web
    - Nodejs
    - wordguess
    - 开源框架
---
# 前言

>业余时间开发的猜词小游戏。



# Goal and Requirements
The application is a game to guess a word.
The user could enter a word,and the page will display the following:
1. if the word is not one of the permitted the words,the app will be able to let them entern a new word.
2.Display the words that the user has guessed .
3.Display how many letters that the word has in common with the word they are trying to guess.


#### server.js
var express = require('express');
var app = express();
const gamePage = require('./public/game')

app.use(express.static('public'));

app.get('/', function (req, res) {
res.send(gamePage.curPage());
})


var server = app.listen(3000, function () {

var host = server.address().address
var port = server.address().port

console.log("Web server is running on http://%s:%s", host, port)

})

#### words.js
讲讲开发中的几个难点
这里需要进行判断输入是否合法，即判断输入的单词是否包含在已定义好的数组中：
isValidInput (input) {
const letters = /^[A-Za-z]+$/;
if (!input || !input.match(letters)) {
return false;
}

input = input.toUpperCase();

if (input.length !== this.answer.length || this.prevGuesses.includes(input)) {
return false;
} else {
return true;
}
}

另外需要考虑到的是在每次选词的时候的随机性，我是这样设计的：
const getRandomIndex = int => {
return Math.floor(Math.random() * int);
};

word比较
const match = (first, second) => {
if (first.length !== second.length) {
return null;
}
first = first.toLowerCase();
second = second.toLowerCase();

const hash = {};

for (let letter of first) {
if (hash[letter]) {
hash[letter] += 1;
} else {
hash[letter] = 1;
}
}

#### Example
If words.js has the words "TEA, EAT, TEE, PEA, PET, APE" and the game selects TEA as the secret word then:

TREE will give a warning about an invalid word, not increment the turn counter and allow a new guess
ATE will give a warning about an invalid word, not increment the turn counter and allow a new guess
PET will respond with 2 matches and increment the turn counter then allow a new guess
TEE will respond with 2 matches and increment the turn counter then allow a new guess
tee will respond with 2 matches and increment the turn counter then allow a new guess
EAT will respond with 3 matches and increment the turn counter then allow a new guess
TEA will respond that they have won the game in however many turns and allow them to start a new game with a new randomly selected word from the list



