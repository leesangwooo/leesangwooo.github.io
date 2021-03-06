---
title: AJAX와 JAVA SERVLET의 데이터 교환
description: AJAX와 JAVA SERVLET의 데이터 교환
categories:
 - labs
tags: java
---

기초학습을 위해 하드코딩 방식으로 소개되었다. 주석한 서블릿의 역할과 스트링버퍼의 사용법 정도만 기억하자.

### AJAX와 JAVA SERVLET의 데이터 교환
AJAX의 비동기 요청으로부터 JAVA SERVLET에서 이를 처리하여 반환 데이터를 다시 AJAX 요청부로 전송하는 과정

### 순서
1. 클라이언트사이드에서 AJAX로 비동기 요청
 : *`$.ajax`* 함수의 속성 설정하기
	- `url` : 요청을 전송할 url

	- `method` : 요청 방식

	- `dataType` : 요청 결과로 반환받을 데이터의 타입
	
	- `success` : 요청과 응답의 과정이 성공적인 경우 호출되는 함수
	
	- `error` : 요청과 응답의 과정이 정상적으로 종료되지 않은 경우 호출되는 함수
	
2. 자바 서블릿에서 요청을 수신하여 ajax로 결과 데이터 반환
	
    1. ajax에서 설정한 반환 데이터의 타입(dataType)을 확인하기 위해 request header의 `accept` 속성값을 확인.
    
    2. `accept` 속성값에 따라 반환 데이터의 표현방식을 변환.(Marshalling)
    
    3. 반환 데이터를 response의 `printWriter.print()' 로 전송 (=JSP 표현식 `<%= %>`)

### CODE SAMPLE
ex) JavaScript (jquery ajax)
````javascript
$.ajax({
    url : "<%=request.getContextPath()%>/prod/getLprodList.do", 
    method : "get",
    dataType : "json",
    success : function(resp) {
        // alert(JSON.stringify(resp)); // 반환(응답) 데이터를 json 형식으로 변환하여 확인
        var html =""; 
        // 반환받은 데이터(resp)는 key와 value의 쌍(Map)들의 집합이다.
        $.each(resp,function(propName, propValue){
            html += ( propValue + ":" + propName );
        });
    },
    error : function(errorResp) {
        alert(errorResp.status);
    }
});

````

ex) Java Servlet (HTTPServlet을 상속받은 java class)
````java
public class LprodAjaxController extends HTTPServlet {

	@Override
	public void doGet(HttpServletRequest req, HttpServletResponse resp) // ajax 요청 방식과 동일한 메소드 오버라이드
			throws ServletException, IOException {

		// ajax에서 설정한 반환 데이터의 타입을 확인하기 위해 request header의 accept 값을 확인한다.
		String accept = req.getHeader("Accept");
		
                // 전송할 데이터
		IOthersDAO othersDAO = new OthersDAOImpl();
		Map<String, Object> lprodMap = othersDAO.selectLprodList(); 
        
		// Marshalling : 데이터 공통 표현방식에 따라 변환하는 작업 (to JSON, XML...)
                //  json 객체형식으로 표현 {"P101":"전자제품","P102":"컴퓨터제품",...}
		StringBuffer json = new StringBuffer("{");
		String ptrn = "\"%s\":\"%s\",";
		for (Entry<String, Object> entry : lprodMap.entrySet()) {
			String propName = entry.getKey();
			Object propValue = entry.getValue();
			json.append(String.format(ptrn, propName, propValue));
		}
		
                // 마지막 ','를 제거
		json.deleteCharAt(json.lastIndexOf(","));
		json.append("}");
		// UnMarshalling 공통 표현방식으로 전달된 데이터를 특정 언어로 변환하는 작업(from JSON, XML)

		// accept 값이 json과 같다면 response의 컨텐츠타입을 아래와 같이 일치시킨다.
		if (StringUtils.containsIgnoreCase(accept, "json")) {
			resp.setContentType("application/json;charset=UTF-8");
			// 직렬화 (Serialize) 전송 작업
			try (PrintWriter out = resp.getWriter();) {
				out.print(json);
			}
		}
	}
````