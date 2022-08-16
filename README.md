# WareHouse

## 개요
대덕인재개발원 최종프로젝트 WareHouse입니다.
WareHouse는 업무처리에 중점을 둔 타 그룹웨어와는 달리 업무처리가 곧 회사의 노하우(KnowHow)가 되도록 분류 및 관리함으로써, 사용자가 편리하게 업무를 파악하고 참여할 수 있도록 돕는 지식형 그룹웨어서비스 입니다.

핵심기능
* 그룹웨어의 주요 기능인 ‘업무’, ‘전자결재’에 해시태그 알고리즘을 도입하여 업무를 진행하거나 새로운 문서를 작성하는 등 사용자가 필요한 순간에 필요한 정보를 자동으로 추천
* 멘토링 관계를 도입하여 공개된 업무와 문서에 대하여 별도의 절차 없이 상호 협력

## 개발환경
* Server : Apache Tomcat 8.5
* BackEnd : Java/Spring FrameWork
* FrontEnd : JavaScript(Jquery), Jsp, BootStrap
* DB : Oracle, MyBatis, SqlDeveloper
* ManageMent : SVN

## 일정 및 설계
### 단위업무
* sitemesh를 이용하여 index페이지와 Iframe페이지 컴포넌트 분리/디자인, 메뉴 이동시 페이지 유지를 위한 함수작성 후 배포
* 메인대시보드
* 업무메뉴
* 전자결재대시보드
* 노하우게시판

### 수행절차
![수행절차](https://user-images.githubusercontent.com/64582209/184789661-588e910a-a4ad-44dd-836f-fa9afac3bc15.PNG)

### ERD
![07 논리ERD](https://user-images.githubusercontent.com/64582209/184789727-962a80f3-1b96-47e2-ab87-a83308aafc93.png)

## UI 및 기능
### 메인대시보드
* 오늘 하루 할 일을 기록하기위한 TodoList 와 빠른업무파악/처리를 목적으로 화면을 설계했습니다.
![메인대시보드](https://user-images.githubusercontent.com/64582209/184790547-796560b9-46b1-4217-9763-103245b57d16.JPG)
* 소스코드
  * [javascript](https://github.com/jangmoonseok/WareHouse/blob/master/src/main/webapp/WEB-INF/views/common/home_js.jsp) 
