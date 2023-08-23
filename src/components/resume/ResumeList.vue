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
    <p>selectedResumeNo: {{ selectedResumeNo }}</p>
    <p>selectedAdNo: {{ selectedAdNo }}</p>
    <p>selectedAdTitle: {{ selectedAdTitle }}</p>
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
    <router link="/dashboard/resume/resumeMgtvue">이력서 작성하러 가기</router>
  </div>
</template>

<script>
export default {
  props: {selectedAdNo: Number, selectedAdTitle: String},
  data() {
    return {
      pageSize: 5,
      currentPage: 1,
      selectedResumeNo: 0,
      modalVisible: false,
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
          this.resumeList = response.data;
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

        const param = {
          adNo: this.selectedAdNo,
          resumeNo: this.selectedResumeNo,
          redirectPageUrl: window.location.href,
          loginId: this.$store.state.loginInfo.loginId,
        };

        console.log(param);

        this.axios
          .post("/resume/submit.do", param)
          .then((response) => {
            if (response.status === 201) {
              console.log(response);
              alert("정상 제출 되었습니다.");

              this.$router.go();
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
<style scoped>
.modal-item {
  background-color: ghostwhite;
}

div#resumeList {
  font-size: 15px;
  border-top: 1px solid darkgray;
  border-left: 1px solid darkgray;
  border-right: 1px solid darkgray;
}

li.resumeElement {
  border-bottom: 1px solid darkgray;
  width: 20em;
  font-size: 15px;
  height: 3em;
  line-height: 3em;
}

li.resumeElement > label {
  display: inline-block;
  margin-bottom: 0.5rem;
  width: 100%;
  text-align: center;
  height: 100%;
}
.submitArea a.close:hover,
a.close:focus {
  cursor: pointer;
}
</style>
