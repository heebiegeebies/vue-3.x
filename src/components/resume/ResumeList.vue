<template>
  <div v-if="hasItems">
    <div>
      <ul>
        <li v-for="r in resumeList" :key="r.resume_no" class="resumeElement">
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
      <a class="btnType gray close" @click.prevent="closeResumeListModal"
        ><span>닫기</span></a
      >
    </div>
  </div>
  <div v-else>
    <p>이력서가 없습니다. 이력서를 작성해주세요</p>
    <a href="http://localhost/resume/resumeMgt.do">이력서 작성하러 가기</a>
  </div>
</template>

<script>
// import {router} from "vue-router";

export default {
  props: {selectedAdNo: Number},
  data() {
    return {
      pageSize: 5,
      currentPage: 1,
      selectedResumeNo: 0,
      modalVisible: false,
      selectedAdTitle: "",
      resumeList: [],
    };
  },
  computed: {
    hasItems() {
      return this.resumeList.length > 0;
    },
  },
  methods: {
    fetchResumeList() {
      const param = new URLSearchParams();

      param.append("pageSize", this.pageSize);
      param.append("cpage", this.currentPage);

      console.log(param);

      this.axios({
        method: "get",
        url: "/resume/resumeListJson.do",
        params: param,
        responseType: "json",
      })
        .then((response) => {
          console.log(response);
          this.resumeList = response.data;
          console.log(this.resumeList); // wrapping with proxy object happens
          console.log(JSON.parse(JSON.stringify(this.resumeList))); // to see a raw content
        })
        .catch(function (error) {
          if (error.response) {
            // The request was made and the server responded with a status code that falls out of the range of 2xx
            const status = error.response.status;
            if (status >= 500) {
              alert("server error");
            } else {
              alert("client error");
            }
          } else if (error.request) {
            // The request was made but no response was received.
            alert("request was made but no response was received");
            console.log(error.request);
          } else {
            // Something happened in setting up the request
            alert("something happened during setting up the request");
            console.log(error.message);
          }
        });
    },
    submit() {
      if (confirm(`${this.selectedAdTitle} 에 지원하시겠습니까?`)) {
        if (!this.selectedAdNo || !this.selectedResumeNo) {
          alert("파라미터가 잘못 설정되었습니다");
          return;
        }

        const redirectPageUrl = window.location.href;

        const param = JSON.stringify({
          adNo: this.selectedAdNo,
          resumeNo: this.selectedResumeNo,
          redirectPageUrl: redirectPageUrl,
        });

        this.axios
          .post("http://localhost/resume/submit.do", param, {
            headers: {"Content-type": "application/json"},
          })
          .then(function (response) {
            if (response.status === 201) {
              console.log(response);
              alert("정상 제출 되었습니다.");
              window.location.href = response.data;
            }
          })
          .catch(function (error) {
            alert(`error occurred ${error.message}`);
          });
      }
    },
  },
  mounted() {
    this.fetchResumeList();
  },
};
</script>
