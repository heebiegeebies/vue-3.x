<template>
  <div id="wrap_area">
    <h2 class="hidden">header 영역</h2>
    <jsp:include page="/WEB-INF/view/common/header.jsp"></jsp:include>

    <h2 class="hidden">컨텐츠 영역</h2>
    <div id="container">
      <ul>
        <li class="lnb">
          <!-- lnb 영역 -->
          <jsp:include page="/WEB-INF/view/common/lnbMenu.jsp"></jsp:include>
          <!--// lnb 영역 -->
        </li>
        <li class="contents">
          <!-- contents -->
          <h3 class="hidden">contents 영역</h3>
          <!-- content -->
          <div class="content">
            <div id="resumeListModal">
              <div id="resumeListModalContainer" v-show="modalVisible">
                <div v-if="resumeList.length === 0">
                  <div>
                    <p>이력서가 없습니다. 이력서를 작성해주세요</p>
                    <a href="http://localhost/resume/resumeMgt.do"
                      >이력서 작성하러 가기</a
                    >
                  </div>
                </div>
                <div v-else id="resumeList">
                  <ul v-for="r in resumeList">
                    <li class="resumeElement">
                      <label class="resumeLabel">
                        {{ r.resume_title }}
                        <input
                          type="radio"
                          name="resumeSelection"
                          :value="r.resume_no"
                          v-model="selectedResumeNo"
                        />
                      </label>
                    </li>
                  </ul>
                </div>
                <div class="submitArea" style="text-align: center">
                  <a class="btnType blue" @click.prevent="submit" name="modal"
                    ><span>지원하기</span></a
                  >
                  <a
                    class="btnType gray close"
                    @click.prevent="closeResumeListModal"
                    ><span>닫기</span></a
                  >
                </div>
              </div>
            </div>
            <p class="Location">
              <a href="../dashboard/dashboard.do" class="btn_set home"
                >메인으로</a
              >
              <span class="btn_nav bold">마이페이지</span>
              <span class="btn_nav bold">최근본공고</span>
              <a href="../system/notice.do" class="btn_set refresh">새로고침</a>
            </p>
            <p class="conTitle">
              <span class="btn_nav bold">최근 본 공고 </span>
            </p>
            <p>최근본 공고는 7일간 저장됩니다.</p>
            <div class="divComGrpCodList">
              <table class="col">
                <caption>
                  caption
                </caption>
                <thead>
                  <tr>
                    <th scope="col">기업명</th>
                    <th scope="col">공고명</th>
                    <th scope="col">고용형태</th>
                    <th scope="col">경력</th>
                    <th scope="col">급여</th>
                    <th scope="col">학력</th>
                    <th scope="col">근무지</th>
                    <th scope="col">직무</th>
                    <th scope="col">마감</th>
                    <th scope="col">기타</th>
                    <th scope="col">기타2</th>
                  </tr>
                </thead>
                <tbody id="historyList">
                  <tr v-for="h in historyList">
                    <td>{{ h.companyName }}</td>
                    <td>
                      <a href="#" @click="openAdDetailModal(h.adNo)">{{
                        h.title
                      }}</a>
                    </td>
                    <td>{{ h.hireType }}</td>
                    <td>{{ h.experience }}</td>
                    <td>{{ h.salary }}</td>
                    <td>{{ h.education }}</td>
                    <td>{{ h.location }}</td>
                    <td>{{ h.position }}</td>
                    <td v-if="h.daysLeft > 0">
                      {{ h.dueDate }}(D- {{ h.daysLeft }} )
                    </td>
                    <td v-else-if="h.daysLeft === 0">오늘마감!</td>
                    <td v-else>마감</td>
                    <td v-if="!h.applied && h.daysLeft >= 0">
                      <button
                        type="button"
                        class="btn blue resumeListButton"
                        @click="openResumeListModal(h.adNo, h.title)"
                      >
                        지원하기
                      </button>
                    </td>
                    <td v-else></td>
                    <td v-if="!h.bookmarked">
                      <button class="bookmarkButton" @click="bookmark(h.adNo)">
                        즐겨찾기
                      </button>
                    </td>
                    <td v-else></td>
                  </tr>
                  <input type="hidden" :value="pagination.totalCount" />
                </tbody>
              </table>
            </div>

            <div class="paging_area" id="historyPagination">
              <ul>
                <li is="paginationTemplate" :currentPage="currentPage">
                  {{ currentPage }}
                </li>
              </ul>
            </div>
          </div>
          <!--// content -->

          <h3 class="hidden">풋터 영역</h3>
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
export default {
  data: function () {
    return {
      currentPage: 1,
      pageSize: 5,
      pageBlockSize: 5,
      keyword: "",
      historyList: [],
      pagination: {},
      paging: "",
      adDetail: {},
      applied: false,
      historyTotalCount: 0,

      resumeList: [],
      currentResumePage: 1,
      selectedResumeNo: 0,
      selectedAdNo: 0,
      modalVisible: false,
      selectedAdTitle: "",
    };
  },
  methods: {
    fetchHistoryList() {
      const param = new URLSearchParams();

      param.append("currentPage", this.currentPage);
      param.append("pageSize", this.pageSize);
      param.append("keyword", this.keyword);

      this.axios
        .get("/history/vueHistoryList.do", param)
        .then(function (response) {
          console.log(response.data);
        });
    },
    fetchListCallback(response) {
      this.historyList = response.data;
      this.pagination = response.pagination;
      const vm = this;

      const totalCount = this.pagination.totalCount;
      
      this.historyTotalCount = totalCount;
      console.log(`total count : ${totalCount}`);

      let pagingHtml = "";

      if (totalCount > 0) {
    	  alert('total count gt 0')
//        pagingHtml = buildPaginationHtml(
//          this.currentPage,
//          totalCount,
//          this.pageSize,
//          this.pageBlockSize,
//          "fetchHistoryList",
//          "fetchListCallback"
//        );
        
      } else {
        const emptyPagingHtml =
          "<div id='emptyResultArea'><p>최근 본 공고가 없습니다.</p></div>";
        pagingHtml = emptyPagingHtml;
      }
      this.paging = pagingHtml;
    },
    openAdDetailModal(adNo) {
      const param = {
        ad_no: adNo,
      };

      const vm = this;

      function adDetailCallback(response) {
        gfModalPop("#adDetailModal");
        historyComponent.applied = vm.checkApplied(response.appliedAt);
        historyComponent.adDetail = response;

        comcombo("company_category", "user_company_category", "sel", response.user_company_category);
        comcombo("resume_experience", "ad_experience", "sel", response.ad_experience);
        comcombo("resume_salary", "ad_salary", "sel", response.ad_salary);
        comcombo("education", "ad_education", "sel", response.ad_education);
        comcombo("position", "ad_position", "sel", response.ad_position);
        comcombo("ad_role", "ad_role", "sel", response.ad_role);
        comcombo("ad_type", "ad_type", "sel", response.ad_type);
        comcombo("location", "ad_location", "sel", response.ad_location);
        comcombo("company_size", "user_company_size", "sel", response.user_company_size);

        disableAllSelectTags();

        function disableAllSelectTags() {
          const parentModal = document.getElementById("adDetailModal");
          const selectTags = parentModal.querySelectorAll("select");

          selectTags.forEach((s) => {
            s.disabled = true;
          });
        }
      }

      callAjax(
        "/ad/adDetailForUser.do",
        "GET",
        "json",
        "false",
        param,
        adDetailCallback
      );
    },
    checkApplied(date) {
      console.log(`appliedAt: ${date}`);
      if (date !== "0" && date !== null && date !== undefined) {
        return true;
      }
      return false;
    },
    fetchResumeList(currentPage, resumeListCallback) {
      currentPage = currentPage || 1;

      const param = {
        pageSize: 10,
        cpage: currentPage,
      };

      callAjax(
        "/resume/resumeListJson.do",
        "GET",
        "json",
        "false",
        param,
        resumeListCallback
      );
    },
    resumeListCallback(response) {
      this.resumeList = response;
    },
    toggleModal() {
      this.modalVisible = !this.modalVisible;
    },
    openResumeListModal(adNo, adTitle) {
      this.selectedAdNo = adNo;
      this.selectedAdTitle = adTitle;
      if (!this.modalVisible) this.modalVisible = true;
    },
    closeResumeListModal() {
      this.selectedAdNo = 0;
      this.selectedResumeNo = 0;
      this.modalVisible = false;
    },
    submit() {
      if (confirm(`${this.selectedAdTitle} 에 지원하시겠습니까?`)) {
        if (!this.selectedAdNo || !this.selectedResumeNo) {
          alert("파라미터가 잘못 설정되었습니다");
          return;
        }
        
        const redirectPageUrl = window.location.href;
        
        const param = JSON.stringify({
        	adNo : this.selectedAdNo,
        	resumeNo: this.selectedResumeNo,
        	redirectPageUrl: redirectPageUrl,
        });
        
        axios
        	.post(API_ROOT + "/resume/submit.do", param, {headers : { 'Content-type': 'application/json'}})
        	.then(function(response){
        		if(response.status === 201){
        			console.log(response);
        			alert('정상 제출 되었습니다.');
        			window.location.href = response.data;
        		};
        	})
        	.catch(function(error) {
        		alert("error occurred");
        	});
      } 
    },
  },
  bookmark(adNo) {
	  if(!adNo) {
		  alert('unexpected error occurred : no adNo present');
		  window.reload();
		  return;
	  }
	  
	  const redirectPageUrl = window.location.href;
	  
	  const param = JSON.stringify({
		    adNo: adNo,
		    redirectPageUrl: redirectPageUrl,
	  });
	  
	  var bookmarkCallback = function(response){
		  console.log(response.data);
		  if(response.status === 200 || response.status === 201){
			  alert('북마크 되었습니다.');
		  }
	  }
	 
	  axios.
	  	post(API_ROOT + "/like/post.do", param, {headers : {'Content-type': 'application/json'}})
	  	.then(function (response) {bookmarkCallback(response)})
	  	.catch(function(error) {alert("error occurred:", error)});
	  
  },
  mounted() {
    this.fetchHistoryList(this.currentPage, this.fetchListCallback);
    this.fetchResumeList(this.currentPage, this.resumeListCallback);
  },
};
</script>
