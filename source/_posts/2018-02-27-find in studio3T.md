---
layout: post
title:  "find in studio3T"
date:   2018-02-27
categories: Node.js
---

- 문제

  - mongo studio3T 에서 {_id: '123'} 로 검색하면 id가 '123' 인 row가 find 되지 않음

- 해결

  - {id:ObjectId('123')}

  ​

​	
