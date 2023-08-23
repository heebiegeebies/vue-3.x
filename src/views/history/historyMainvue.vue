<template>
  <div id="wrap_area">
    <h2 class="hidden">header 영역</h2>
    <h2 class="hidden">컨텐츠 영역</h2>
    <div id="container">
      <ul>
        <!-- <li class="lnb"></li> -->
        <li class="contents">
          <!-- contents -->
          <h3 class="hidden">contents 영역</h3>
          <!-- content -->
          <div class="content">
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
            <div>
              <div class="divComGrpCodList" v-if="hasItems">
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
                    <tr v-for="h in historyList" :key="h.historyNo">
                      <td>
                        <input type="hidden" :value="h.historyNo" />
                        {{ h.companyName }}
                      </td>
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
                        <button
                          class="bookmarkButton"
                          @click="bookmark(h.adNo)"
                        >
                          즐겨찾기
                        </button>
                      </td>
                      <td v-else></td>
                    </tr>
                  </tbody>
                </table>
                <div>
                  <paginate
                    class="justify-content-center"
                    v-model="pagination.currentPage"
                    :page-count="pageBlockSize"
                    :page-range="pageBlockSize"
                    :margin-pages="0"
                    :prev-text="'Prev'"
                    :next-text="'Next'"
                    :container-class="`pagination`"
                    :page-class="`page-item`"
                    :click-handler="clickCallback"
                  ></paginate>
                </div>
              </div>
              <p v-else>{{ emptyListMessage }}</p>
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
import {openModal} from "jenesius-vue-modal";
import ResumeList from "@/components/resume/ResumeList.vue";
import Paginate from "vuejs-paginate-next";

export default {
  data: function () {
    return {
      pageBlockSize: 0,
      keyword: "",
      historyList: [],
      pagination: {pageSize: 5, currentPage: 1, totalCount: 0},
      adDetail: {},
      applied: false,
      historyTotalCount: 0,
      emptyListMessage: "목록이 없습니다.",

      currentResumePage: 1,
      selectedResumeNo: 0,
      selectedAdNo: 0,
      modalVisible: false,
      selectedAdTitle: "",
    };
  },
  computed: {
    hasItems() {
      return this.historyList.length > 0;
    },
  },
  methods: {
    calculatePage() {
      const total = this.pagination.totalCount;
      const page = this.pagination.pageSize;
      const xx = total % page;
      let result = parseInt(total / page);

      if (xx == 0) {
        this.pageBlockSize = result;
      } else {
        result = result + 1;
        this.pageBlockSize = result;
      }

      console.log(`page count: ${result}`);
    },
    fetchHistoryList() {
      const param = new URLSearchParams();

      param.append("currentPage", this.pagination.currentPage);
      param.append("pageSize", this.pagination.pageSize);
      param.append("keyword", this.keyword);
      // param.append("loginId", this.$store.state.loginInfo.loginId);

      this.axios
        .get("/history/vueHistoryList.do", {
          params: param,
        })
        .then((response) => {
          this.historyList = response.data.data;
          this.pagination = response.data.pagination;
          this.calculatePage();
        })
        .catch((error) => {
          alert(`error occurred: ${error.message}`);
        });
    },
    // openAdDetailModal(adNo) {
    //   const param = {
    //     ad_no: adNo,
    //   };

    //   const vm = this;

    //   function adDetailCallback(response) {
    //     gfModalPop("#adDetailModal");
    //     historyComponent.applied = vm.checkApplied(response.appliedAt);
    //     historyComponent.adDetail = response;

    //     comcombo(
    //       "company_category",
    //       "user_company_category",
    //       "sel",
    //       response.user_company_category,
    //     );
    //     comcombo(
    //       "resume_experience",
    //       "ad_experience",
    //       "sel",
    //       response.ad_experience,
    //     );
    //     comcombo("resume_salary", "ad_salary", "sel", response.ad_salary);
    //     comcombo("education", "ad_education", "sel", response.ad_education);
    //     comcombo("position", "ad_position", "sel", response.ad_position);
    //     comcombo("ad_role", "ad_role", "sel", response.ad_role);
    //     comcombo("ad_type", "ad_type", "sel", response.ad_type);
    //     comcombo("location", "ad_location", "sel", response.ad_location);
    //     comcombo(
    //       "company_size",
    //       "user_company_size",
    //       "sel",
    //       response.user_company_size,
    //     );

    //     disableAllSelectTags();

    //     function disableAllSelectTags() {
    //       const parentModal = document.getElementById("adDetailModal");
    //       const selectTags = parentModal.querySelectorAll("select");

    //       selectTags.forEach((s) => {
    //         s.disabled = true;
    //       });
    //     }
    //   }

    //   callAjax(
    //     "/ad/adDetailForUser.do",
    //     "GET",
    //     "json",
    //     "false",
    //     param,
    //     adDetailCallback,
    //   );
    // },
    clickCallback(pageNum) {
      this.pagination.currentPage = pageNum;
      this.fetchHistoryList();
    },
    checkApplied(date) {
      console.log(`appliedAt: ${date}`);
      if (date !== "0" && date !== null && date !== undefined) {
        return true;
      }
      return false;
    },
    openResumeListModal: async function (adNo, adTitle) {
      const props = {
        selectedAdNo: adNo,
        selectedAdTitle: adTitle,
      };

      const modal = await openModal(ResumeList, props);

      modal.onclose = () => {
        console.log("close");
        return true;
      };
    },
    bookmark(adNo) {
      if (!adNo) {
        alert("예외 발생: adNo가 입력되지 않았습니다");
        return;
      }

      const param = {
        adNo: adNo,
        loginId: this.$store.state.loginInfo.loginId,
      };

      this.axios
        .post("/like/post.do", param)
        .then((response) => {
          if (response.status === 200 || response.status === 201) {
            alert("정상적으로 스크랩 되었습니다.");
            this.$router.go();
          }
        })
        .catch(function (error) {
          alert("에러 발생", error);
        });
    },
  },
  mounted() {
    this.fetchHistoryList();
  },
  components: {
    paginate: Paginate,
  },
};
</script>
