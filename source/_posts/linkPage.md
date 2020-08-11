---
title: linkPage
categories:
  - etc
tags:
  - link
date: 2020-08-11 14:11:25
thumbnail:
---

## 바로가기

``` bash
<!doctype html>
<html lang="ko">
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>바로가기</title>
	
	
	<style type="text/css">
	*{margin:0;padding:0;font-family: "Malgun Gothic",-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;box-sizing:border-box;}
	ul, li{list-style: none;}
	a{text-decoration: none;}
	body{background:#f2f4f4;}
	header{padding:25px 20px;background:#232f3e;color:#fff;}
	header h1{font-size:40px;}
	
	.select_section{background:#fff;padding:10px 20px;font-size:18px;}
	.select_section select{width:300px;padding:5px;font-size:15px;}
	
	.detail_section{padding:20px;}
	.detail_section > div{margin-top:25px;}
	.detail_section > div:first-child{margin-top:0;}
	.detail_section h2{font-size:28px;font-weight:400;line-height:40px;padding:5px 0;}
	.detail_section ul{display:flex;flex-wrap:wrap;margin:0 auto;}
	.detail_section ul li{padding:10px;display:flex;width:25%;}
	.detail_section ul li a{display:block;width:100%;border-top:5px solid #232f3e;background:#fff;box-shadow: 0 1px 3px 0 rgba(0,0,0,.3),0 0 0 1px rgba(0,0,0,.04);text-align:center;padding:20px;font-size:17px;font-weight:700;color:#007eb9;}
	.detail_section ul li a:hover{outline:0;color:#e47911;text-decoration:underline;}
	</style>

</head>
<body>

	<!-- #wrap -->
	<div id="wrap">
		<!-- #header -->
		<header>
			<h1>즐겨찾기</h1>
		</header>
		<!-- // #header -->

		<!-- #container -->
		<div id="container">
			<section class="select_section">
				<label for="gubun">서버구분 : </label>
				<select id="gubun">
					<option value="https://test2.bemyunicorn.io">테스트2   : https://naver.com</option>
					<option value="https://test3.bemyunicorn.io">테스트3   : https://daum.net</option>
					<option value="https://bemyunicorn.io">운영   : https://google.com</option>
				</select>
			</section>

			<section class="detail_section">
				<!-- 사이트 -->
				<div>
					<h2>사이트</h2>
					<ul>
						<li>
							<a href="javascript:window.open(document.getElementById('gubun').value + '/login/ompAdmLogin');">메인</a>
						</li>
					</ul>
				</div>
				<!-- // 사이트 -->

				<!-- API 테스트페이지 -->
				<div>
					<h2>API 테스트페이지</h2>
					<ul>
						<li>
							<a href="javascript:window.open(document.getElementById('gubun').value + '/');">블라블라</a>
						</li>					
					</ul>
				</div>
				<!-- // API 테스트페이지 -->
				
				<!-- 연동사이트 -->
				<div>
					<h2>연동사이트</h2>
					<ul>
						<li>
							<a href="javascript:window.open('https://docs.sendbird.com');">센드버드 DOC</a>
						</li>
					</ul>
				</div>
				<!-- // 연동사이트 -->
			</section>
		</div>
		<!-- // #container -->
	</div>
	
</body>
</html>
```