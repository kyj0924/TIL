# ajax를 활용한 게시판 만들기(22.11.24 ncs 시험)

## 디렉토리

![Untitled](ajax를_활용한_게시판_만들기(22_11_24_ncs_시험)_defcdadbc9f74f6190a70410eff35dc4/Untitled.png)

### BoardController.java

```java
package kr.co.ezen.controller;

import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import kr.co.ezen.entity.Member;
import kr.co.ezen.service.BoardService;

@Controller
public class BoardController {
	
	@Autowired
	private BoardService boardService;
	
	@RequestMapping("/")
	public String main() {
		return "redirect:main";
	}
	
	@GetMapping("/main")
	public String mainG() {
		return "main";
	}
	
	@PostMapping("/loginPro")
	public String loginPro(@RequestParam String id, @RequestParam String pwd, Model mo, HttpSession session) {
		Member mem = boardService.loginCheck(id);
		
		if(mem!=null && mem.getPwd().equals(pwd)) {
			mo.addAttribute("id", id);
			session.setAttribute("loginMember", mem);
			return "loginSuccess";
		}
		
		return "loginFail";
	}
	
	@PostMapping("/join")
	public String join() {
		return "join";
	}
	
	@PostMapping("/joinPro")
	public String joinPro(Member vo) {
		if(vo.getAddress1()!="") {
			String addr = vo.getAddress1()+vo.getAddress2()+vo.getAddress3()+vo.getAddress4();
			vo.setAddress(addr);
		}
		
		if(vo.getZipcode2()!="") {
			String zipc = vo.getZipcode2();
			vo.setZipcode(zipc);
		}
		

		boardService.insertMember(vo);
		return "main";
	}
	
	@GetMapping("/cancelPro")
	public String cancelPro() {
		return "redirect:main";
	}
	
	@PostMapping("/logoutPro")
	public String logoutPro(HttpSession session) {
		// 세션 종료
		session.invalidate();
		return "redirect:main";
	}
	
	@RequestMapping("/memberInfo")
	public @ResponseBody List<Member> memberInfo(HttpSession session, Member mem){
		
		List<Member> li = new ArrayList<Member>();
		mem = (Member)session.getAttribute("loginMember");
		li.add(mem);		
		return li;
	}
	/*
	@RequestMapping("/checkId")
	public @ResponseBody int checkId(@RequestParam("id") String id){
		
		int countId = boardService.checkIdDup(id);
		System.out.println(countId);
		return countId;
	}
	*/
	@PostMapping("/checkDuplicateId")
	   @ResponseBody
	   public String checkDuplicateId(@RequestParam("id") String id) {
	       boolean isDuplicate = boardService.isIdDuplicated(id);
	       return isDuplicate ? "duplicate" : "available";
	   }

}
```

### Member.java

```java
package kr.co.ezen.entity;

import lombok.Data;

@Data
public class Member {
	
	private String id;
	private String pwd;
	private String pwd2;
	private String name;
	private String gender;
	private String email;
	private String birth;
	
	private String zipcode;
	
	// api로 입력받은 우편번호
	private String zipcode2;
	
	private String address;
	
	// api로 입력받은 주소
	private String address1;
	private String address2;
	private String address3;
	private String address4;
	
	private String hobby;
	private String job;

}
```

### db.sql

```sql
create table member(
	id varchar(20),
	pwd varchar(20) not null,
	name varchar(20) not null,
	gender char(1) not null,
	email varchar(20) not null,
	birth varchar(6) not null,
	zipcode char(5),
	address varchar(50),
	hobby char(5),
	job varchar(20),
	primary key(id)
);

drop table member;

select * from member;

select id from Member where id='aaa';

create table tblzipcode(
	zipcode char(5),
	area1 char(20),
	area2 varchar(20),
	area3 char(40),
	primary key(zipcode)
);
```

### BoardMapper.java

```java
package kr.co.ezen.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;

import kr.co.ezen.entity.Member;

@Mapper
public interface BoardMapper {
	
	public Member loginCheck(String id);
	public void insertMember(Member vo);
	//public int checkIdDup(String id);
	int countById(String id);

}
```

### BoardMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="kr.co.ezen.mapper.BoardMapper">
	<!-- 로그인을 위한 DB에 저장되어 있는 id, pw 확인 -->
	<select id="loginCheck" resultType="kr.co.ezen.entity.Member" parameterType="String">
		select * from Member where id=#{id};		
	</select>
	
	<!-- 회원가입하면 정보가 DB에 저장된다 -->
	<insert id="insertMember" parameterType="kr.co.ezen.entity.Member">
		insert into Member values (#{id},#{pwd},#{name},#{gender},#{email},#{birth},#{zipcode},#{address},#{hobby},#{job}) 
	</insert>
	
	<!-- 아이디 중복체크 -->
	<!-- 
	<select id="checkIdDup" resultType="int" parameterType="String">
		select count(id) from Member where id=#{id};
	</select>
	-->
	<select id="countById" resultType="int" parameterType="String">
        SELECT COUNT(*) FROM member WHERE id = #{id}
    </select>
	
</mapper>
```

### BoardService.java

```java
package kr.co.ezen.service;

import kr.co.ezen.entity.Member;

public interface BoardService {
	
	public Member loginCheck(String id);
	public void insertMember(Member vo);
	//public int checkIdDup(String id);
	boolean isIdDuplicated(String id);

}
```

### BoardServiceImpl.java

```java
package kr.co.ezen.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import kr.co.ezen.entity.Member;
import kr.co.ezen.mapper.BoardMapper;

@Service
public class BoardServiceImpl implements BoardService{
	
	@Autowired
	private BoardMapper boardMapper;
	
	@Override
	public Member loginCheck(String id) {
		
		return boardMapper.loginCheck(id);
	}

	@Override
	public void insertMember(Member vo) {
		boardMapper.insertMember(vo);
	}
	/*
	@Override
	public int checkIdDup(String id) {
		return boardMapper.checkIdDup(id);
		
	}
	*/
	@Override
    public boolean isIdDuplicated(String id) {
		int count = boardMapper.countById(id);
        return count > 0;
   }

	
	
}
```

### join.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
   pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<link rel="stylesheet"
   href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
<script
   src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
<script
   src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>

<script src="//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"></script>
<title>Insert title here</title>
<style type="text/css">
body {
   height: 100vh;
   display: flex;
   align-items: center;
   justify-content: center;
}

.form-group {
   background-color: rgb(224, 224, 224);
   padding: 10px;
   border-radius: 20px;
}
</style>
</head>
<body>
   
   
   
   
   <form id="fr">
      <div class="form-group">
         <h1>회원가입</h1>
         <table>
            <tr height="50">
               <td width="150">아이디*</td>
               <td width="350" align="center"><input type="text"
                  class="form-control" id="id" name="id"></td>
               <td width="150" align="center">
                  <button type="button" class="btn btn-success" name="idcheck" id="idCheck_btn">ID중복확인</button>
               </td>
            </tr>

            <tr height="50">
               <td width="150">비밀번호*</td>
               <td width="350" align="center"><input type="password"
                  class="form-control" name="pwd" size="40"></td>
            </tr>

            <tr height="50">
               <td width="150">비밀번호 확인*</td>
               <td width="350" align="center"><input type="password"
                  class="form-control" name="pwd2" size="40"></td>
            </tr>

            <tr height="50">
               <td width="150">이름*</td>
               <td width="350" align="center"><input type="text"
                  class="form-control" name="name" size="40"></td>
            </tr>
            <div class="radio">
               <tr height="50">
                  <td width="150">성별*</td>
                  <td width="350" align="center"><label><input
                        type="radio" name="gender" value="m">남성</label> <label><input
                        type="radio" name="gender" value="f">여성</label></td>
               </tr>
            </div>

            <tr height="50">
               <td width="150">생년월일*</td>
               <td width="350" align="center"><input type="text"
                  class="form-control" name="birth" size="22"></td>
               <td width="150">ex)900323</td>
            </tr>

            <tr height="50">
               <td width="150">E-mail*</td>
               <td width="350" align="center"><input type="email"
                  class="form-control" name="email" size="40"></td>
            </tr>

            <tr height="50" class="addr1">
               <td width="150">우편번호</td>
               <td width="350" align="center">
               <input type="text" class="form-control" name="zipcode" size="22" placeholder="직접입력">
               </td>
               <td width="150" align="center">
               <input type="button" onclick="sample4_execDaumPostcode()" class="btn btn-warning" value="우편번호 찾기">
               </td>
            </tr>
            
            <tr class="addr2">
            	<td width="150">우편번호</td>
            	<td>
               <input type="text" class="form-control" name="zipcode2" id="sample4_postcode" placeholder="우편번호">
            	</td>
            	<td width="150" align="center">
            	<input type="button" onclick="sample4_execDaumPostcode()" class="btn btn-warning" value="우편번호 찾기">
            	</td>
            </tr>

            <tr height="50" class="addr1">
               <td width="150">주소</td>
               <td width="350" align="center">
               <input type="text" class="form-control" name="address" size="40">
               </td>
            </tr>
            
            <tr height="50" class="addr2">
               <td width="150">주소</td>
               <td>
             	<input type="text" class="form-control" name="address1" id="sample4_roadAddress"placeholder="도로명주소">
               	<input type="text" class="form-control" name="address2" id="sample4_jibunAddress" placeholder="지번주소">
               	<input type="text" class="form-control" name="address3" id="sample4_detailAddress" placeholder="상세주소">
               	<input type="text" class="form-control" name="address4" id="sample4_extraAddress" placeholder="참고항목">
               </td>
            </tr>

            <tr height="50">
               <td width="150">취미</td>
               <td width="350" align="center"><label class="checkbox-inline">
                     <input type="checkbox" name="hobby" size="40" value="인터넷">인터넷
               </label> <label class="checkbox-inline"><input type="checkbox"
                     name="hobby" size="40" value="여행">여행</label> <label
                  class="checkbox-inline"><input type="checkbox"
                     name="hobby" size="40" value="게임">게임</label> <label
                  class="checkbox-inline"><input type="checkbox"
                     name="hobby" size="40" value="영화">영화</label> <label
                  class="checkbox-inline"><input type="checkbox"
                     name="hobby" size="40" value="운동">운동</label></td>
            </tr>

            <tr height="50">
               <td width="150">직업</td>
               <td align="center"><select name="job">
                     <option value="전문직">전문직</option>
                     <option value="학생">학생</option>
                     <option value="선생">선생</option>
                     <option value="회사원">회사원</option>
               </select></td>
            <tr height="50">
               <td width="150" align="center" colspan="3">
                  <button type="button" class="btn btn-success" id="joinSuc_btn">회원가입</button>
                  <input type="reset" class="btn btn-danger" value="다시쓰기"
                  id="reset_btn">
                  <button type="button" class="btn btn-warning" id="cancel_btn">취소</button>
               </td>
         </table>
      </div>
   </form>
   <script >
	$('.addr2').hide();
   $(document).ready(function(){
      $('#joinSuc_btn').click(function(){
         $('#fr').attr("action", "joinPro");
         $('#fr').attr("method", "post");
         $('#fr').submit();
      });
      
      $('#cancel_btn').click(function(){
         $('#fr').attr("action", "cancelPro");
         $('#fr').attr("method", "get");
         $('#fr').submit();
      })
      
   })
   
   
   //여기서부터는 카카오 맵 API구역 입니다.
   
   //본 예제에서는 도로명 주소 표기 방식에 대한 법령에 따라, 내려오는 데이터를 조합하여 올바른 주소를 구성하는 방법을 설명합니다.
    function sample4_execDaumPostcode() {
        new daum.Postcode({
            oncomplete: function(data) {
                // 팝업에서 검색결과 항목을 클릭했을때 실행할 코드를 작성하는 부분.

                // 도로명 주소의 노출 규칙에 따라 주소를 표시한다.
                // 내려오는 변수가 값이 없는 경우엔 공백('')값을 가지므로, 이를 참고하여 분기 한다.
                var roadAddr = data.roadAddress; // 도로명 주소 변수
                var extraRoadAddr = ''; // 참고 항목 변수

                // 법정동명이 있을 경우 추가한다. (법정리는 제외)
                // 법정동의 경우 마지막 문자가 "동/로/가"로 끝난다.
                if(data.bname !== '' && /[동|로|가]$/g.test(data.bname)){
                    extraRoadAddr += data.bname;
                }
                // 건물명이 있고, 공동주택일 경우 추가한다.
                if(data.buildingName !== '' && data.apartment === 'Y'){
                   extraRoadAddr += (extraRoadAddr !== '' ? ', ' + data.buildingName : data.buildingName);
                }
                // 표시할 참고항목이 있을 경우, 괄호까지 추가한 최종 문자열을 만든다.
                if(extraRoadAddr !== ''){
                    extraRoadAddr = ' (' + extraRoadAddr + ')';
                }

                // 우편번호와 주소 정보를 해당 필드에 넣는다.
                document.getElementById('sample4_postcode').value = data.zonecode;
                document.getElementById("sample4_roadAddress").value = roadAddr;
                document.getElementById("sample4_jibunAddress").value = data.jibunAddress;
                
                // 참고항목 문자열이 있을 경우 해당 필드에 넣는다.
                if(roadAddr !== ''){
                    document.getElementById("sample4_extraAddress").value = extraRoadAddr;
                } else {
                    document.getElementById("sample4_extraAddress").value = '';
                }
                $('.addr1').hide();
                $('.addr2').show();
            }
        }).open();
    }
   
   
   
   $('#idCheck_btn').click(function(){
	   var id = $("#id").val();
	   if(id.length==0){
		   alert('아이디를 입력하세요');
	   }
	   else{

	// 입력된 ID를 서버로 전송하여 중복 여부 확인
       $.ajax({
          url: "checkDuplicateId",
          type: "POST",
          data: { "id": id },
          success: function(response) {
             if (response === "available") {
                alert("사용 가능한 아이디입니다.");
             } else {
                alert("이미 사용 중인 아이디입니다. 다른 아이디를 입력해주세요.");
             }
          },
          error: function() {
             alert("중복 체크 중 오류가 발생했습니다.");
          }
       });
	   }
    });

   
   
</script>
</body>
</html>
```

### loginFail.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<link rel="stylesheet"
	href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css">
<script
	src="https://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script
	src="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
	<script type="text/javascript">
		alert("id와 password를 다시 확인하세요");
		history.go(-1);
	</script>
</body>
</html>
```

### loginSuccess.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<link rel="stylesheet"
	href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css">
<script
	src="https://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script
	src="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/js/bootstrap.min.js"></script>

<script>
	$(document).ready(function(){
		waitMemberInfo();
	})

	
	function waitMemberInfo(){
		$.ajax({
			url:"memberInfo",
			type:"post",
			dataType:"json",
			success:createView,
			error:function(){
				alert("error");
			}
		});
	}
	
	function createView(data){
		var list="<form id='frm'>";
		list+="<h2>My Info</h2>";
		list+="<table class='table table-bordered'>";
		list+="<tr>";
		list+="<td>아이디</td>";
		list+="<td>이름</td>";
		list+="<td>성별</td>";
		list+="<td>이메일</td>";
		list+="<td>생년월일</td>";
		list+="<td>우편번호</td>";
		list+="<td>주소</td>";
		list+="<td>취미</td>";
		list+="<td>직업</td>";
		list+="</tr>";
		$.each(data, function(index, obj){
			list+="<tr>";
			list+="<td>"+obj.id+"</td>";
			list+="<td>"+obj.name+"</td>";
			list+="<td>"+obj.gender+"</td>";
			list+="<td>"+obj.email+"</td>";
			list+="<td>"+obj.birth+"</td>";
			list+="<td>"+obj.zipcode+"</td>";
			list+="<td>"+obj.address+"</td>";
			list+="<td>"+obj.hobby+"</td>";
			list+="<td>"+obj.job+"</td>";
			list+="</tr>";
		});
		list+="<button type='button' class='btn btn-primary btn-md' id='logout_btn'>로그아웃</button>";
		list+="</table>";
		list+="</form>";
		
		// panel-body의 Content 부분에 지금 데이터 출력하는 코드 한 줄
		$('#view').html(list);
		$('#view').css("display", "block");
		
		
		$('#logout_btn').click(function(){
			alert('로그아웃 되었습니다');
			$('#frm').attr("action", "logoutPro");
			$('#frm').attr("method", "post");
			$('#frm').submit();
		});
	}

	
	
</script>

</head>
<body>
	<script>
		alert("${id}님 환영합니다.");
		/*
		$('#logout_btn').click(function(){
			alert('로그아웃 되었습니다');
			$('#frm').attr("action", "logoutPro");
			$('#frm').attr("method", "post");
			$('#frm').submit();
		});
		*/
	</script>
	
<div class="container">
      <h2>My Info</h2>
      <div class="panel panel-default">
         <div class="panel-heading">${id }님의 정보</div>
         <div class="panel-body" id="view">Content</div>
      </div>
</div>	
	
</body>
</html>
```

### main.jsp

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<link rel="stylesheet"
	href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css">
<script
	src="https://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script
	src="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/js/bootstrap.min.js"></script>
<style>
	body{
	height: 100vh;
	display: flex;
	justify-content: center;
	align-items: center;
	}
	
	.container{
	background-color: gray;
	width: 400px;
	height: 300px;
	border-radius: 20px;
	}
</style>
</head>
<body>

	<script>
		$(document).ready(function(){
			$('#login_btn').click(function(){
				$('#fr').attr("action", "loginPro");
				$('#fr').attr("method", "post");
				$('#fr').submit();	
			});
			
			$('#join_btn').click(function(){
				$('#fr').attr("action", "join");
				$('#fr').attr("method", "post");
				$('#fr').submit();
			})
			
			
		})
	</script>

	<div class="container">
		<h2>사용자 로그인</h2>
		<form id="fr">
			<div class="form-group">
				<label for="id">ID:</label> <input type="text" class="form-control"
					id="id" placeholder="Enter id" name="id">
			</div>
			<div class="form-group">
				<label for="pwd">Password:</label> <input type="password"
					class="form-control" id="pwd" placeholder="Enter password"
					name="pwd">
			</div>
			<button type="button" class="btn btn-success" id="login_btn">로그인</button>
			<button type="button" class="btn btn-primary" id="join_btn">회원가입</button>
		</form>
	</div>
</body>
</html>
```

### servlet-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
	<context:component-scan base-package="kr.co.ezen.controller" />
	
	
	
</beans:beans>
```

### root-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
	<!-- service는 mapper보다 상단에 올려줘야 함 -->
	<context:component-scan base-package="kr.co.ezen.service" />
	
	<!-- Root Context: defines shared resources visible to all other web components -->
	<bean class="com.zaxxer.hikari.HikariConfig" id="hikariConfig">
		<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/com?useSSL=false"/>
		<property name="username" value="com"/>
		<property name="password" value="com01"/>	
	</bean>
	
	<bean class="com.zaxxer.hikari.HikariDataSource" id="dataSource" destroy-method="close">
		<constructor-arg ref="hikariConfig"/>	
	</bean>
	
	<bean class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	
	<mybatis-spring:scan base-package="kr.co.ezen.mapper"/>
	
	
</beans>
```

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="https://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
		
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

</web-app>
```

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>kr.co</groupId>
	<artifactId>ezen</artifactId>
	<name>AWS1102</name>
	<packaging>war</packaging>
	<version>1.0.0-BUILD-SNAPSHOT</version>
	<properties>
		<java-version>1.8</java-version>
		<org.springframework-version>5.0.7.RELEASE</org.springframework-version>
		<org.aspectj-version>1.6.10</org.aspectj-version>
		<org.slf4j-version>1.6.6</org.slf4j-version>
	</properties>
	<dependencies>

		<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>2.0.1.Final</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator -->
		<dependency>
			<groupId>org.hibernate.validator</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>6.2.0.Final</version>
		</dependency>

		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework-version}</version>
			<exclusions>
				<!-- Exclude Commons Logging in favor of SLF4j -->
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>

		<!-- AspectJ -->
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>${org.aspectj-version}</version>
		</dependency>

		<!-- Logging -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${org.slf4j-version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>jcl-over-slf4j</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${org.slf4j-version}</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
			<exclusions>
				<exclusion>
					<groupId>javax.mail</groupId>
					<artifactId>mail</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.jms</groupId>
					<artifactId>jms</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jdmk</groupId>
					<artifactId>jmxtools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jmx</groupId>
					<artifactId>jmxri</artifactId>
				</exclusion>
			</exclusions>
			<scope>runtime</scope>
		</dependency>

		<!-- @Inject -->
		<dependency>
			<groupId>javax.inject</groupId>
			<artifactId>javax.inject</artifactId>
			<version>1</version>
		</dependency>

		<!-- Servlet -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.49</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>5.0.7.RELEASE</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
		<dependency>
			<groupId>com.zaxxer</groupId>
			<artifactId>HikariCP</artifactId>
			<version>3.3.1</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.6</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.3.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.12</version>
			<scope>provided</scope>
		</dependency>

		<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jacksondatabind -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.9.8</version>
		</dependency>

		<!-- Test -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<artifactId>maven-eclipse-plugin</artifactId>
				<version>2.9</version>
				<configuration>
					<additionalProjectnatures>
						<projectnature>org.springframework.ide.eclipse.core.springnature</projectnature>
					</additionalProjectnatures>
					<additionalBuildcommands>
						<buildcommand>org.springframework.ide.eclipse.core.springbuilder</buildcommand>
					</additionalBuildcommands>
					<downloadSources>true</downloadSources>
					<downloadJavadocs>true</downloadJavadocs>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.5.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<compilerArgument>-Xlint:all</compilerArgument>
					<showWarnings>true</showWarnings>
					<showDeprecation>true</showDeprecation>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>exec-maven-plugin</artifactId>
				<version>1.2.1</version>
				<configuration>
					<mainClass>org.test.int1.Main</mainClass>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```