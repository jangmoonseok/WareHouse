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

## UI 및 기능(핵심기능의 핵심로직을 소스코드 첨부했습니다.)
### 메인대시보드
* 오늘 하루 할 일을 기록하기위한 TodoList 와 빠른업무파악/처리를 목적으로 화면을 설계했습니다.
![메인대시보드](https://user-images.githubusercontent.com/64582209/184804912-79168611-4cea-46fa-a814-75544221d639.JPG)


### 업무대시보드
* 업무의 마감일을 지키게할 목적으로 화면을 설계했습니다.
![업무대시보드](https://user-images.githubusercontent.com/64582209/184805935-faad39ba-a050-43b9-906c-cc95b32f91e6.JPG)
* 핵심기능
  * 금주 마감 업무
  ```javascript
  //JavaScript 소스코드
  //금주 마감 업무(내업무) 페이지 목록 가져오기
  function getThisWeekEndMyWorkList(page){
  //탭 이동 처리
		$('.home_tab li').removeClass("active");
		$('.myWork').addClass("active")
		workListPage = page;
		$.ajax({
			url : "<%=request.getContextPath()%>/work/getThisWeekEndMyWorkList.do?page=" + page,
			type : 'get',
			success : function(res){
				if(res.thisWeekEndMyWorkList.length == 0){
    //결과가 없을때
					var str = `
						<td colspan="6" style="text-align:center;">
							<span>금주 마감 업무가 존재하지 않습니다.</span>
						</td>
					`;
					$('.workList').html(str)
				}else{
    //결과를 Handlebars를 사용해 후처리 
					printWorkList(res.thisWeekEndMyWorkList, $('.workList'), $('#workList-template'));
				}
    //pageMaker객체를 이용해 목록 페이지 구현
				printPageMaker(res.pageMaker,$('#workList_pagination'), $('#myWorkList-pagination-template') )
				$('.myWork').addClass("active");
				$('.toReq').removeClass("active");
			},
			error : function(error){
   //에러 경고를 SweetAlert라이브러리를 이용해 디자인
				Swal.fire({
				      icon: 'error',
				      title: "error : " + error.status,
				      confirmButtonColor: '#3085d6',
				    });
			}
		});
	}
  //Handlebars template
  <script type="text/x-handlebars-template"  id="workList-template">
    {{#each .}}
     <tr style="color:{{readCheckColor wcheck}}; font-size:14px; cursor:pointer" onclick="detail_go('{{wstatus}}', '{{wcode}}')">
      <td>
       <span style="font-weight:{{readCheckFontWeight wcheck}}
        text-overflow:ellipsis; overflow:hidden; width:80%; display:inline-block; white-space:nowrap">{{wtitle }}</span>
       {{workOverDay wstatus overDay}}
      </td>
      <td>
       <div style="align-items: center; display: flex;">
        <div>
         <img class="table-avatar emp_profile" src="{{requesterPhoto}}">
        </div>
        <div>
         <span style="font-size:12px; font-weight:bold">{{requester }}</span>
        </div>
       </div>
      </td>
      <td>
       <div style="align-items: center; display: flex;">
          <div>
           <img class="table-avatar emp_profile" src="{{managerPhoto}}">
          </div>
          <div>
           <span style="font-size:12px; font-weight:bold">{{manager}}</span>

          </div>
          <div style="font-size: 12px; ">
           <span style="font-size: 8px; margin-left: 5px">외 {{managerCnt}}명</span>
          </div>
       </div>
      </td>
      <td class="project_progress">
       <div class="progress progress-sm">
        <div class="progress-bar bg-green" role="progressbar" aria-valuenow="{{wprogress}}" aria-valuemin="0"
        aria-valuemax="100" style="width: {{wprogress}}%"></div>
       </div>
       <small>{{wprogress}}% 완료</small>
      </td>
      <td>
       {{dateFormat wend}}
       {{workDday wstatus dDay}}
      </td>
      <td>
       <span class="badge {{workStatus wstatus}}">
        {{wstatus}}
       </span>
      </td>
     </tr>
    {{/each}}
   </script>
  ```

  ```java
  //Java Service 소스코드
  public Map<String, Object> getThisWeekToReqList(int eno, Criteria cri) throws SQLException {
   Map<String, Object> dataMap = new HashMap<String, Object>();

   cri.setPerPageNum(5); // 한 페이지에 보여줄 목록 개수 설정

   List<WorkVO> thisWeekEndToReqList = workDAO.selectThisWeekEndToReqList(eno, cri); // mybatis를 통해 목록결과 가져오기

   thisWeekEndToReqList = setWorkInfo(thisWeekEndToReqList); 		// 업무담당자,요청자,마감일 기준 남은 일수 계산 등 업무의 세부정보 셋팅
   setWorkReadCheck(thisWeekEndToReqList, eno); // 읽은업무 체크

   int totalCount = workDAO.selectThisWeekEndToReqTotalCount(eno); //pagination을 위한 총 갯수 설정
   PageMaker pageMaker = new PageMaker();
   pageMaker.setCri(cri);
   pageMaker.setTotalCount(totalCount);

   dataMap.put("thisWeekEndToReqList", thisWeekEndToReqList);
   dataMap.put("pageMaker", pageMaker);

   return dataMap;
  }
  ```

### 업무목록
* 업무의 상태별/읽지않은 업무를 한 눈에 확인할 수 있게 화면을 설계했습니다.
![업무목록](https://user-images.githubusercontent.com/64582209/184812408-e4c50436-a1f7-4281-a3e8-b78c6d923ac1.JPG)
* 핵심기능
  * 상태별 업무목록
  ```javascript
   //JavaScript 소스코드
   function list_go(url, page, statusNo){
   //url : 이동할 url
   //page : 이동할 page
   //statusNo : 이동할 업무상태번호
   
   //Form태그에 parameter값 설정
   var jobForm=$('#jobForm');
   jobForm.find("[name='page']").val(page);
   jobForm.find("[name='perPageNum']").val();
   jobForm.find("[name='searchType']").val($('select[name="searchType"]').val());
   jobForm.find("[name='keyword']").val($('input[name="keyword"]').val());
   jobForm.find("[name='statusNo']").val(statusNo);
   jobForm.find("[name='mCode']").val();



   jobForm.attr({
    action:url,
    method:'get'
   }).submit();
  }
  ```
 
  ```java
  //Java Service 소스코드
  /** cri : 검색,페이지설정정보를 담은 객체
	 *  statusNo : 업무상태를 나타내는 고유번호
	 *  eno : 로그인한 유저 기본키
	 */
	@Override
	public Map<String, Object> getMyWorkList(Criteria cri, int statusNo, int eno) throws SQLException {
		//업무상태 고유번호를 문자열 업무상태로 변경
		String wstatus = statusNo == 0 ? "대기" : statusNo == 1 ? "진행" : statusNo == 2 ? "완료" : statusNo == 3 ? "협업요청" :
			statusNo == 4 ? "대리요청" : "전체";

		Map<String, Object> dataMap = new HashMap<String, Object>();

		//workList
		List<WorkVO> myWorkList = null;
		int totalCount = 0;
		//업무의 상태가 전체이면 전체업무목록 가져오고 아니면 해당 상태의 목록만 가져오기
		if(wstatus.equals("전체")) {
			myWorkList = workDAO.selectMyWorkAllStatusList(eno, cri);
		}else {
			myWorkList = workDAO.selectMyWorkList(eno,wstatus,cri);
		}
		//총 페이지를 표시하기위한 업무상태에 따른 총 결과 갯수
		totalCount = workDAO.selectMyWorkTotalCount(eno, wstatus,cri);

		//pageMaker
		PageMaker pageMaker = new PageMaker();
		pageMaker.setCri(cri);
		pageMaker.setTotalCount(totalCount);

		// 업무담당자,요청자,마감일 기준 남은 일수 계산 등 업무의 세부정보 셋팅
		myWorkList = setWorkInfo(myWorkList);
		setWorkReadCheck(myWorkList, eno);
		for(WorkVO work : myWorkList) {
			//업무의 해시태그 셋팅
			String hashTag = workDAO.selectHashTagByWcode(work.getWcode());
			work.setHashTag(hashTag);
		}

  
		dataMap.put("myWorkList", myWorkList);
		dataMap.put("pageMaker", pageMaker);
		dataMap.put("statusNo", statusNo);

		return dataMap;
	}
  ```
### 업무 상세페이지
* 대기업무는 업무를 승인,이의신청,부당신고가능
* 진행업무는 업무수정, 협업요청, 대리요청가능하며 그 외 업무보고서, 회의록, 질의응답 등 효율적인 업무처리 목적으로 화면을 설계했습니다.
* 상세페이지는 공통적으로 관련노하우와 멘토의 관련업무를 목록으로 나타내 업무를 진행하면서 많은 레퍼런스를 참조할 수 있게 하였습니다.
![미승인업무상세](https://user-images.githubusercontent.com/64582209/184814370-4d18a0cf-1f5d-454e-8298-7ee70d4dc22b.JPG)
![진행중업무상세](https://user-images.githubusercontent.com/64582209/184814601-c0565d77-f93c-4eb0-b3d5-363d0fec37ff.JPG)
* 핵심기능
  * 관련 노하우
  ```javascript
  //JavaScript 소스코드
  //관련 노하우 가져오기
	function getRelationKnowhow(page){
		//page : 이동할 페이지

		var data;
		if(!$('input[name="rnKeyword"]').val()){
			//검색어가 없다면 기본추천
			data = {
				wcode : '${work.wcode}',
				page : page
			}
			url = "<%=request.getContextPath()%>/work/getRelationKnowhow.do";
		}else{
			//검색어가 있을경우 검색어와 관련된 노하우 추천
			data = {
				page : page,
				searchType : 'h',
				keyword: $('input[name="rnKeyword"]').val()
			}
			url = "<%=request.getContextPath()%>/kw/knowhow/getAllKnowHowList.do";
		}


		$.ajax({
			url : url,
			type : 'post',
			data : data,
			success : function(res){
				if(res.knowhowList.length == 0){
    //결과가 없을때
					var str = `
						<tr>
							<td colspan="2">
								관련 노하우가 존재하지 않습니다.
							</td>
						</tr>
					`;

					$('.relationKnowhow').html(str);
				}else{
    //결과를 Handlebars를 사용해 후처리
					printList(res.knowhowList, $('.relationKnowhow'), $('#relation-template'));
				}
				relationKnowhowPage = page;
    //pageMaker객체를 이용해 목록 페이지 구현
				printRelationPageMaker(res.pageMaker, $('#relationKnowhowPagination'), $('#relationKnowhow-paination-template'));
			},
			error : function(error){
				AjaxErrorSecurityRedirectHandler(error.status);
			}

		});
	}
  ```
  
  ```xml
  //관련 노하우 SQL쿼리문
  <select id="selectRecommendWorkList" resultType="work">
		select distinct g.wcode, g.wtitle, g.wdate, g.wend, g.wopen,walarm, g.eno, g.wprogress, g.wstatus, g.classcode, g.wcontent, h.viewcnt
		  from hashtag f, work g, knowhow h,
		       (select distinct b.tagcontent
		          from(select regexp_substr(a.langlist, '[^ ]+', 1, level) as tagcontent
		                 from (select tagcontent as langlist
		                         from hashtag
							    where classcode = 'B103') a
							    <![CDATA[
			   		  connect by level <= length(regexp_replace(a.langlist, '[^ ]+','')) + 1) b
			   		  ]]>
				 where not b.tagcontent like '%'||'년차'||'%'

				intersect

				select distinct d.tagcontent
				  from(select regexp_substr(c.tagcontent, '[^ ]+', 1, level) as tagcontent
		                 from (select b.tagcontent
		                         from work a, hashtag b
				        where a.wcode = b.hashno
						  and a.wcode = #{wcode}
						  and a.wstatus !='완료') c
						     <![CDATA[
					  connect by level <= length(regexp_replace(c.tagcontent, '[^ ]+','')) + 1) d
					   ]]>
				 where not d.tagcontent like '%'||'년차'||'%'
				 ) e
		 where f.hashno = g.wcode
		   and g.wcode = h.wcode
		   and f.tagcontent like '%'||e.tagcontent||'%'
			 <if test="keyword != ''.toString()">
				 and f.tagcontent like '%'||#{keyword}||'%'
			 </if>
		 order by h.viewcnt desc
	</select>
  ```

### 지시한 업무 수정페이지
* 내가 지시한 업무는 제목을 제외한 업무의 모든 정보를 수정할 수 있게 설계했습니다.
* jstree 라이브러리를 이용해 Restful방식으로 조직도를 구현해 담당자를 추가/삭제할 수 있습니다.
![요청한업무수정](https://user-images.githubusercontent.com/64582209/184819561-25a32bb9-e9ee-4945-bfcb-3b699864c3a7.JPG)
* 핵심기능
  * 조직도를 통한 담당자 추가
  ```javascript
  //JavaScript 소스코드
  //jstree 생성함수
		$('#organization').jstree({
			core : {
				data :${organizationNode} // 조직도 노드 목록
			},
			types : {
				'default' : {'icon': 'jstree-folder'}
			},
			 plugins: ['wholerow', 'types']
		})
		
		//조직도 클릭 이벤트 함수
		 $('#organization').on("changed.jstree", function (e, data) {
		    if(data.node.id.length > 3){
		    	//클릭한 조직도의 노드가 사원일때 사원정보를 가져와 담당자 추가하기
		    	$.ajax({
		    		url : "<%=request.getContextPath()%>/work/getEmpByNodeId.do?eno=" + data.node.id,
		    		type:'get',
		    		success:function(res){
		    			if(res.eno == ${work.eno}){
		    				//클릭한 사원이 업무 요청자일때
		    				Swal.fire({
		  				      icon: 'warning',
		  				      title: '업무 요청자는 담당자로 추가할 수 없습니다.',
		  				      confirmButtonColor: '#3085d6',
		  				    });
		    			}else if($('div[data-eno="' + res.eno + '"]').length == 0){
		    				//새로운 담당자를 추가했을때
		    				if('${work.wstatus}' == '대리요청' && newWorkManager.length >= 1){
		    					//업무의 상태가 대리요청이고 이미 담당자를 추가했을때
		    					Swal.fire({
				  				      icon: 'warning',
				  				      title: '이미 담당자를 수정하였습니다.',
				  				      confirmButtonColor: '#3085d6',
				  				    });
		    				}else if('${work.wstatus}' == '진행'){
		    					//업무의 상태가 대리요청,협업요청도 아닌 진행상태에서는 담당자를 추가할 수 없다.
				    			Swal.fire({
				  				      icon: 'warning',
				  				      title: '담당자를 추가할 수 없습니다.',
				  				      confirmButtonColor: '#3085d6',
				  				    });
		    				}else{
		    					//담당자 추가
				    			addEmp(res, $('.emp_List'), $('#addEmp-template'))
				    			var hashTag = $('input[name=hashTag]').val();

			    				var tagList = hashTag.split(" ");
			    				console.log(tagList);
			    				var hashCodeSet = new Set(tagList);
			    				hashCodeSet.add("#" + res.year + "년차");
			    				console.log(hashCodeSet);

			    				var result = "";
			    				for(var code of hashCodeSet){
			    					result += code+" ";
			    				}

			    				$('input[name=hashTag]').val(result.trim());

				    			addWorkManager(res.eno);
		    				}
		    			}else{
		    				// 클릭한 사원이 이미 추가된 담당자일때
		    				Swal.fire({
			  				      icon: 'warning',
			  				      title: '이미 등록된 담당자는 추가할 수 없습니다.',
			  				      confirmButtonColor: '#3085d6',
			  				    });
		    			}

		    		},
		    		error:function(error){
		    			alert(error);
		    		}
		    	});
		    }
	 	 });
  ```
  ```java
  //조직도 node구성을 위한 Java소스코드
  /**
	 * depList : 조직도를 구성하기 위한 부서VO 목록
	 */
	public List<OrganizationNode> organization(List<DepartmentVO> depList){
		List<OrganizationNode> nodeList = new ArrayList<OrganizationNode>();

		//부서의 고유 노드ID, 부서명, 부서에 속한 사원 설정
		for(DepartmentVO dep : depList) {
			OrganizationNode node = new OrganizationNode();
			node.setId(Integer.toString(dep.getDno()));
			node.setText(dep.getDname());
			if(dep.getUpdno() == 0) {
				node.setParent("#");
			}else {
				node.setParent(Integer.toString(dep.getUpdno()));
			}

			if(dep.getEmpList().size() > 0) {
				List<EmployeeVO> empList = dep.getEmpList();
				for(EmployeeVO emp : empList) {
					OrganizationNode childNode = new OrganizationNode();
					childNode.setId(Integer.toString(emp.getEno()));
					childNode.setText(emp.getName());
					childNode.setParent(Integer.toString(dep.getDno()));
					childNode.setIcon("fas fa-user");
					nodeList.add(childNode);

				}
			}

			node.setIcon("fas fa-building");

			nodeList.add(node);
		}

		return nodeList;
	}
 
  //Node정보를 담은 OrganizationNode객체
  public class OrganizationNode {

  String id;
  String parent;
  String text;
  List<OrganizationNode> children;
  String icon;



  public String getIcon() {
   return icon;
  }
  public void setIcon(String icon) {
   this.icon = icon;
  }
  public List<OrganizationNode> getChildren() {
   return children;
  }
  public void setChildren(List<OrganizationNode> children) {
   this.children = children;
  }
  public String getId() {
   return id;
  }
  public void setId(String id) {
   this.id = id;
  }
  public String getParent() {
   return parent;
  }
  public void setParent(String parent) {
   this.parent = parent;
  }
  public String getText() {
   return text;
  }
  public void setText(String text) {
   this.text = text;
  }
  @Override
  public String toString() {
   return ToStringBuilder.reflectionToString(this, ToStringStyle.JSON_STYLE);
  }


  }
  ```
  
### 노하우게시판
* 모든 완료된 업무가 공개상태라면 노하우가 되어 노하우게시판에서 조회가 가능합니다.
* 내가 원하는 노하우를 쉽게 찾기위해 내 업무에 등록된 해시태그를 클릭 시 관련된 노하우만 출력되도록 설계했습니다.
![노하우게시판](https://user-images.githubusercontent.com/64582209/184822617-bab0ab2b-d63b-4431-97bb-d84a72299bc1.JPG)
* 핵심기능
  * 해시태그 클릭 시 자동 검색결과 출력
  ```javascript
  //JavaScript 소스코드
  //노하우 목록 가져오기
	function getAllKnowhowList(page, type, content){
		//page : 이동할 페이지
		//type : 검색유형
		//content : 검색어
		var allKnowHowPage = page;
		var searchType = $('select[name="searchType"]').val();
		var keyword = $('input[name="keyword"]').val();
		if(type){
			//검색유형이 선택됐을때 검색에 필요한 parameter설정
			searchType = type,
			keyword = content
		}
		$.ajax({
			url : "<%=request.getContextPath()%>/kw/knowhow/getAllKnowHowList.do",
			type : 'get',
			data : {
				page : page,
				searchType : searchType,
				keyword: keyword
			},
			success:function(res){
				if(res.knowhowList.length == 0){
					var str = `
						<table class="table table-hover text-nowrap"
						style="text-align: center; table-layout: fixed;">
						<tr style="font-size: 0.95em;">
							<th style="width: 30%; text-align:center;">제목</th>
							<th style="width: 20%; text-align:center;">요청자</th>
							<th style="width: 30%; text-align:center;">담당자</th>
							<th style="width: 10%; text-align:center;">조회수</th>
							<th style="width: 10%; text-align:center;">즐겨찾기</th>
						</tr>
						<tr>
							<td colspan="5" style="text-align:center">
								노하우가 존재하지 않습니다.
							</td>
						</tr>
						</table>
					`
					$('.knowhowList').html(str);
				}else{
					printWorkData(res.knowhowList, $('.knowhowList'), $('#recommendWorkList-template'));
				}
				//공통 pagination template를 사용하기 위한 pageMaker객체 값 설정
				lPage = page - 1;
				if(lPage < 1){lPage = 1;}
				rPage = page + 1;
				if(rPage > res.pageMaker.realEndPage){rPage = res.pageMaker.realEndPage;}
				pageMakerData = {
						pageMaker : res.pageMaker,
						target : 'getAllKnowhowList',
						left : "javascript:getAllKnowhowList("+lPage+")",
						right : "javascript:getAllKnowhowList("+rPage+")",
						doubleLeft : "javascript:getAllKnowhowList("+(1)+")",
						doubleRight :"javascript:getAllKnowhowList("+(res.pageMaker.endPage)+")"
				}
				$('input[name="searchType"]').val(res.pageMaker.cri.searchType);
				$('input[name="keyword"]').val(res.pageMaker.cri.keyword);

				printPageMaker(pageMakerData ,$('#knowhowlistPage'), $('#pageMaker-template'));
			},
			error:function(error){
				AjaxErrorSecurityRedirectHandler(error.status);
			}
		});
	}
  ```
  
### 전자결재 대시보드
* 업무대시보드와 다르지 않게 빠른 결재처리와 결재가 누락되는것을 방지할 수 있도록 화면을 설계했습니다.
![진행중업무상세](https://user-images.githubusercontent.com/64582209/184824544-f5914c6a-c005-45f0-a5c4-b3f942452df9.JPG)
