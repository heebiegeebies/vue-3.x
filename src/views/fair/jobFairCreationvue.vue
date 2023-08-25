<template>
  <form id="frm" name="frm">
    <div id="wrap_area">
      <h2 class="hidden">header 영역</h2>
      <h2 class="hidden">컨텐츠 영역</h2>
      <div id="container">
        <ul>
          <li class="contents">
            <!-- contents -->
            <h3 class="hidden">contents 영역</h3>
            <!-- content -->
            <div class="content">
              <p class="Location">
                <a href="../dashboard/dashboard.do" class="btn_set home"
                  >메인으로</a
                >
                <span class="btn_nav bold">취업박람회</span>
                <span class="btn_nav bold">박람회등록 </span>
                <a href="../fair/jobFairCreation.do" class="btn_set refresh"
                  >새로고침</a
                >
              </p>
              <p class="conTitle">
                <span class="btn_nav bold">박람회 등록 </span>
              </p>
              <br />
              <br />
              <h4>박람회 정보</h4>
              <br />
              <table class="row" style="margin-top: 0.5%">
                <colgroup>
                  <col width="12%" />
                  <col width="12%" />
                  <col width="12%" />
                  <col width="12%" />
                  <col width="12%" />
                  <col width="12%" />
                </colgroup>

                <tbody>
                  <tr>
                    <th scope="row">박람회명</th>
                    <td>
                      <input
                        type="text"
                        class="inputTxt p100"
                        name="fairname"
                        id="fairname"
                        v-model="formData.fairname"
                      />
                      <p>output: {{ formData.fairname }}</p>
                    </td>
                  </tr>
                  <tr>
                    <th scope="row">주소</th>
                    <td>
                      <input
                        type="text"
                        class="inputTxt"
                        style="width: 50%"
                        name="zip_code"
                        id="zip_code"
                        v-model="formData.zip_code"
                        readonly
                      />
                      <input
                        type="button"
                        value="우편번호 찾기"
                        @click="execDaumPostcode"
                        id="post_cd"
                        style="width: 35%; heigh: 50%; cursor: pointer"
                      />
                      <input
                        type="text"
                        class="inputTxt"
                        style="width: 90%"
                        name="addr"
                        id="addr"
                        v-model="formData.addr"
                        readonly
                      />
                      <input
                        type="text"
                        class="inputTxt"
                        style="width: 90%"
                        name="detAddr"
                        id="detAddr"
                        v-model="formData.detAddr"
                      />
                      <p>
                        zipcode: {{ formData.zip_code }} addr:
                        {{ formData.addr }} detAddr: {{ formData.detAddr }}
                      </p>
                    </td>
                  </tr>
                  <tr>
                    <th scope="row">이미지첨부</th>
                    <td>
                      <input
                        multiple="multiple"
                        type="file"
                        class="btnType blue"
                        name="addfile"
                        id="addfile"
                        @change="handleFileChange"
                      />
                      <p>{{ file.name }}</p>
                    </td>
                  </tr>
                  <tr>
                    <th scope="row">시작일자</th>
                    <td>
                      <input
                        type="date"
                        class="datepicker"
                        placeholder="0000-00-00"
                        name="startdate"
                        id="startdate"
                        v-model="formData.startdate"
                        :min="datepicker.minOfStartDate"
                        @change="setMinOfEndDate(formData.startdate)"
                      />
                      <p>{{ formData.startdate }}</p>
                    </td>
                  </tr>
                  <template v-if="formData.startdate">
                    <tr>
                      <th scope="row">종료일자</th>
                      <td>
                        <input
                          type="date"
                          class="datepicker"
                          placeholder="0000-00-00"
                          name="enddate"
                          id="enddate"
                          v-model="formData.enddate"
                          :min="datepicker.minOfEndDate"
                        />
                        <p>{{ formData.enddate }}</p>
                      </td>
                    </tr>
                  </template>
                  <tr>
                    <th scope="row">전체참여인원수</th>
                    <td>
                      <input
                        type="text"
                        class="inputTxt p100"
                        name="userNum"
                        id="userNum"
                        v-model="formData.userNum"
                        placeholder=""
                        @input="checkNumberInput(formData.userNum)"
                      />
                      <p>{{ formData.userNum }}</p>
                      <p v-if="numberInputMessage">{{ numberInputMessage }}</p>
                    </td>
                  </tr>
                  <tr>
                    <th scope="row">전시장 선택</th>
                    <td>
                      <select
                        id="location"
                        name="location"
                        style="width: 65%"
                        v-model="formData.booth"
                      >
                        <template v-for="option in location" :key="option.no">
                          <option :value="option.value">
                            {{ option.name }}
                          </option>
                        </template>
                      </select>
                    </td>
                  </tr>
                </tbody>
              </table>
              <img id="location_image" /> <br />
              <div class="btn_areaC mt30" id="updateBtnArea">
                <a class="btnType blue" @click="validate">
                  <span>등록하기</span>
                </a>
              </div>
            </div>
            <!--// content -->
            <h3 class="hidden">풋터 영역</h3>
          </li>
        </ul>
      </div>
    </div>
    <div>
      <div>
        <template v-if="!daumPostcode.isOpen">
          <a class="button" @click="openDaumPostcodeModal">우편번호입력</a>
        </template>
        <template v-else>
          <VueDaumPostcode @complete="onComplete">
            <template #loading>
              <div>...Loading...</div>
            </template>
          </VueDaumPostcode>
        </template>
      </div>
      <div>
        <pre v-if="daumPostcode.result">{{ daumPostcode.result }}</pre>
      </div>
    </div>
  </form>
</template>

<script>
import {openModal} from "jenesius-vue-modal";
import {VueDaumPostcode} from "vue-daum-postcode";

export default {
  data() {
    return {
      daumPostcode: {
        result: null,
        isOpen: false,
        addr: "",
        detAddr: "",
      },
      formData: {
        fairname: "",
        zip_code: "",
        addr: "",
        detAddr: "",
        startdate: "",
        enddate: "",
        booth: 0,
        userNum: "",
      },
      file: {
        name: "",
        content: null,
      },
      numberInputMessage: "",
      numberRegex: /\D/g,

      location: [
        {value: 0, name: "선택"},
        {value: 1, name: "전시장A"},
        {value: 2, name: "전시장B"},
        {value: 3, name: "전시장C"},
      ],
      datepicker: {
        date: new Date(),
        minOfStartDate: "",
        minOfEndDate: "",
      },
      validationRules: [
        {prop: "formData.fairname", message: "박람회명을 입력해주세요"},
        {prop: "formData.zip_code", message: "주소를 확인해주세요"},
        {prop: "formData.addr", message: "주소를 확인해주세요"},
        {prop: "formData.detAddr", message: "주소를 확인해주세요"},
        {
          prop: "formData.startdate",
          message: "시작일 또는 종료일을 입력하세요",
        },
        {prop: "formData.enddate", message: "시작일 또는 종료일을 입력하세요"},
        {prop: "formData.userNum", message: "박람회 수용인원을 입력해주세요"},
        {prop: "formData.locationValue", message: "전시장을 선택해주세요"},
        {prop: "file.name", message: "이미지를 첨부해주세요"},
        {prop: "file.content", message: "이미지를 첨부해주세요"},
      ],
    };
  },
  methods: {
    openDaumPostcodeModal: async function () {
      const modal = await openModal(VueDaumPostcode);
      modal.onclose = () => {
        return true;
      };
    },
    onComplete(newResult) {
      this.daumPostcode.result = newResult;
      this.daumPostcode.isOpen = false;
      this.formData.zip_code = newResult.zonecode;
      this.formData.addr = newResult.addr;
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      this.file.content = file;
      this.file.name = file.name;
    },
    submit() {
      console.log(this.formData);
      console.log(this.file);

      // const data = this.formData;

      // const param = {
      //   data: data,
      //   file: this.file.content,
      // };
    },
    validate() {
      const rules = this.validationRules;
      const vm = this;

      for (const rule of rules) {
        console.log(`prop: ${rule.prop}`);
        const value = getPropertyValue(rule.prop);

        if (!value) {
          alert(rule.message);
          return;
        }
      }

      function getPropertyValue(propPath) {
        const props = propPath.split(".");
        let value = vm;

        for (const prop of props) {
          value = value[prop];

          console.log(`value: ${value}`);

          if (!value) {
            break;
          }
        }
        return value;
      }

      alert("유효성검사 통과");

      this.submit();
    },
    checkNumberInput(input) {
      // 문자열 입력시 안내 메시지
      if (this.numberRegex.test(input)) {
        this.numberInputMessage = "숫자만 입력가능합니다.";
        // 숫자 외의 문자 제거
        this.formData.userNum = input.replace(this.numberRegex, "");
      } else {
        this.numberInputMessage = "";
      }
    },
    setMinOfEndDate(minOfStartDate) {
      const originalDate = minOfStartDate;
      if (!originalDate) {
        alert("시작일자가 선택되지 않았습니다.");
        return;
      }

      this.datepicker.minOfStartDate = minOfStartDate;

      // 기존 날짜로 새로운 Date 생성
      const newDate = new Date(originalDate);

      // 날짜 변경
      newDate.setDate(newDate.getDate() + 1);

      const formattedDate = this.createFormattedDayAsYYYYmmDD(newDate);
      this.datepicker.minOfEndDate = formattedDate;

      this.formData.enddate = "";
    },
    createFormattedDayAsYYYYmmDD(dateToConvert) {
      if (!(dateToConvert instanceof Date)) {
        console.log("파라미터는 Date 타입이어야 합니다.");
        return;
      }

      const year = dateToConvert.getFullYear();
      const month = String(dateToConvert.getMonth() + 1).padStart(2, "0");
      const day = String(dateToConvert.getDate()).padStart(2, "0");
      const formattedDate = `${year}-${month}-${day}`;

      return formattedDate;
    },
  },
  mounted() {
    const date = this.datepicker.date;
    this.datepicker.minOfStartDate = this.createFormattedDayAsYYYYmmDD(date);
  },
};
</script>
