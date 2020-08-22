---
layout: post
title: Làm thế nào để lấy dữ liệu từ web trường ?
subtitle: Section 1 của series "Tôi có thể làm gì với web trường ?"
tags: [codedao]
comments: true
---

# Chuẩn bị

## Kiến thức:

Cái gì cũng thế trước khi học hay làm gì thì ta cũng cần có chút kiến thức nền tảng (cơ mà đi học mấy môn đại cương ở trường tôi thấy mình như tờ giấy trắng 😭😭😭 )

- **[Puppeteer là gì ? Có ăn được không ?](https://github.com/puppeteer/puppeteer/blob/main/README.md)** **Mình thấy series trên khá cơ bản và dễ tìm hiểu với các bạn mới tiếp xúc nhé**
  - [Làm quen với puppeteer](https://toidicodedao.com/2017/12/12/puppeteer-headless-chrome-api-phan-1/)
  - [Thực hành với puppeteer](https://toidicodedao.com/2017/12/19/puppeteer-headless-chrome-api-phan-2-cao-du-lieu-kenh14/)

**Puppeteer** là một thư viện rất mạnh mẽ nhưng cũng có vô số API =.= Nhìn mà muốn rối loạn tiền đình luôn 😰 . May mắn là document khá chi tiết và dễ đọc nên cũng được phần nào 🤗🤗🤗 Chứ cứ như ông React (sắp tới sẽ làm việc với thằng này ở Section tiếp) thì có mà mù luôn.

- API của Puppeteer mà chúng ta cần quan tâm
  - **puppeteer.launch**: Mở trình duyệt Chrome lên để bắt đầu làm trò. Hàm này trả về object kiểu Browser.
  - **browser.newPage**: Mở một tab mới trong Chrome để làm trò. Hàm này trả về object kiểu Page.
  - **browser.close**: Tắt trình duyệt (Đỡ phải tắt bằng tay)
  - **page.goto** : Đi tới một trang nào đó. Có params waitUntil khá quan trọng. Params này quyết định chúng ta chờ tới khi page vừa mới load xong, hay sau khi page đã load toàn bộ JavaScript và hình ảnh.
  - **page.screenshot**: Chụp ảnh tab hiện tại, lưu thành file ảnh.
  - **page.evaluate**: Đây là API quan trọng nhất, cho phép ta chạy script trong browser và lấy kết quả trả về.
  - **page.type** : Cho phép chúng ta nhập dữ liệu vào input. Xử lý cho đăng nhập, search ,...
  - **page.click** : Dùng để click vào các nút đã đăng nhập

## Hành trang :

Với một thân kungfu , ta con phải chuẩn bị ý kẹo bánh mạng đi học nào 🍞 🍖 🍕

Không thể thiếu rồi !!! Đầu tiên là chọn editor. Có nhiều loại lắm, tùy bạn quen dùng loại nào thôi. Còn nếu pro cứ sài notepad nhé :V

- Editor :
  - [Visual Studio Code](https://code.visualstudio.com/) (recommended) : free, nhẹ, ngon
  - [Atom](https://atom.io/) : free, ngon, có vẻ hơi nặng
  - [Sublime Text](https://www.sublimetext.com/): nhẹ
  - Vân Vân và Mây Mây :v
- Môi trường : [Node.js](https://nodejs.org/en/) các bạn cứ tải và cài đặt bản LTS cho ổn định nhé, bạn nào dùng bản cũ mà chưa support [async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) thì update lên nhé
  - [Bất đồng bộ trong JavaScript - Callback, Promise đến Observable](https://www.youtube.com/watch?v=qW3raOCefms) Video này của anh Tiep Phan nói rất chi tiết nhé

# Let's Go ! Giờ bắt tay vào phá cái web trường nào .

**Web site trường sử dụng:** http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Home.aspx

Với các trang web của trường khác sử dụng mã nguồn của CMC thì các bạn cũng có thể làm giống nhé fix tí xíu thui hihi 😜😜😜

## Khởi tạo Project

1. **Các bạn chạy lện bên dưới để khởi tạo thư mục gốc của project và int npm nhé**

```
mkdir student-api
cd student-api
npm init -y
```

2.  **Cài đặt thư viện cần thiết cho project**

```
npm i --save puppeteer moment moment-timezone lodash ava
```

3. **Cấu trúc thư mục project sẽ như sau**

- root
  - pages : lưu code chạy cho từng page
  - util : các thư viện hỗ trợ
  - selectors : các selector của các element tác động tới
  - test : Dùng để test code

4. **Khởi tạo khung code cho project**

- Tạo file _index.js_ trong **root** :

```
// Require dependencies
const puppeteer = require("puppeteer");

// Require custom lib

// Main

class StudentAPI {
  // Khởi tạo
  constructor(opts = {}) {
    this._opts = opts;
    this._user = null;
  }

  // Kiểm tra user có tồn tại không => boolean
  get isAuthenticated() {
    return !!this._user;
  }
  // Trả về user
  get User() {
    return this._user;
  }

  // Khởi chạy browser
  async Browser() {
    // Nếu browser đã tồn tại thì trả về cái cũ , nếu không khởi tạo browser mới
    if (!this._browser) {
      this._browser =
        this._opts.browser || (await puppeteer.launch(this._opts.puppeteer));
    }
    return this._browser;
  }

  // Đóng browser khi không còn dùng
  async Close() {
    const browser = await this.Browser();
    await browser.close();

    this._browser = null;
    this._user = null;
  }
}
module.exports = StudentAPI;
```

Bây giờ vào thư mục **test** thêm file _index.js_

```
const test = require("ava");
const StudentAPI = require("../");

test("basic", (t) => {
  const api = new StudentAPI();
  t.is(api.isAuthenticated, false);
  t.is(api.user, null);
  console.log(api.isAuthenticated, api.user);
});
```

Chạy `npm run test`, Well done. Kết quả đúng như mong đợi

```
1 tests passed
```

## Nào giờ cùng đào bới xem web trường có gì ???

Về logic xử lý : đầu tiên chúng ta sẽ phải vào trang đăng nhập, sau đó vào đến trang có lịch học vào cào thôi. Nghe đã thấy dễ rồi chúng ta bắt tay vào làm nhé

## _Xử lý login_

1. Trước hết chúng ta xem qua cấu trúc của cái trang đăng nhập.

![Trang Login](img/student-api/login.png)

Ta dễ dàng nhận thấy một số selector của **_input_** Tài khoản và Mật khẩu, **_Button_** Đăng nhập lần lượt là

```
  #txtUserName
  #txtPassword
  #btnSubmit
```

Tạo file _LoginSelector.js_ trong thư mục **selectors** :

```
module.exports = {
  account: "#txtUserName",
  password: "#txtPassword",
  btnLogin: "#btnSubmit",
  userRole: "#PageHeader1_lblUserRole"
};
```

Bây giờ chúng ta sẽ dùng API goto, type, click của Puppeteer để đăng nhập nhé

Tạo file _Login.js_ trong **pages** :

```
const {
  account,
  password,
  btnLogin,
  userRole,
} = require("../selectors/LoginSelector");

module.exports = async (browser, { idUser, passwordUser } = user) => {
  // Tạo page mới
  const page = await browser.newPage();
  // Truy cập vào trang đăng nhập
  await page.goto("http://dkh.tlu.edu.vn/cmcsoft.iu.web.info/Login.aspx");
  // Điền thông tin đăng nhập vào trong input
  await page.type(account, idUser);
  await page.type(password, passwordUser);
  // Click button đăng nhập
  await page.click(btnLogin);
  await page.waitForSelector(userRole);

  // Check xem có đăng nhập thành công hay không ?
  const element = await page.$(userRole);
  const text = await (await element.getProperty("textContent")).jsonValue();
  if (!text.length) throw new Error("Lỗi đăng nhập");
  await page.close();
};

```

Bây giờ vào **index.js** update thêm hàm thôi nào:

```
...

// Require custom lib
const LoginPage = require("./pages/Login");
// Main
class StudentAPI {
  ...
  // Hàm đăng nhập
  async Login(user) {
    const browser = await this.Browser();
    // Nếu không phát sinh lỗi (đăng nhập thành công) thì this._user sẽ tồn tại
    try {
      await LoginPage(browser, user);
      this._user = user;
    } catch (error) {
      throw new Error(error.message);
    }
  }
  ...
}
module.exports = StudentAPI;

```

Giờ chúng ta vào thư mục **test** thêm vào _index.js_ , nhớ sửa tài khoản mật khẩu của bạn nhé

```
....
test("authenicated", async (t) => {
  const api = new StudentAPI();
  const user = { idUser: "tài khoản", passwordUser: "mật khẩu" };
  await api.Login(user);
  t.is(api.isAuthenticated, true);
  t.is(api.user, user);
});
```

Kết quả :v Well done !!! Chạy ngon lành

```
2 tests passed
```

Yeahhhhhhhhhhhhh !!!🤟🤟🤟 Chỉ với dòng code ta đã có thể đăng nhập tự động thành công rùi 😜😜😜. Browser chúng ta khởi tạo lúc này sẽ lưu lại cookie và giờ chúng ta sẽ vào trang lịch học để cào thôi.

## _Xử lý thời gian học_

Sau khi vào được trang: http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Reports/Form/StudentTimeTable.aspx

Chúng ta cũng mở console lên để code thử lấy dữ liệu nào

```
 // Lưu lại dữ liệu về lịch học các môn
    let timeTable = [];

    //Regex bóc dữ liệu ngày tháng và thời gian học
    const data_pattern = /(.+?) đến (.+?):/;
    const time_pattern = /Thứ ([0-9]) tiết ([0-9,]+?) \((.+?)\)/;
    const location_pattern = /\(([0-9])+\)\n(.+)/g;

    // Bóc dữ liệu html lịch học từng môn
    let timeTableHTML = [
      ...document.querySelectorAll("#gridRegistered tr"),
    ].slice(1, 11);

    for (let html of timeTableHTML) {
      let subjectHTML = [...html.querySelectorAll("td")];

      // Chuyển dữ liệu từng môn hoc về dạng text
      let [
        STT,
        lopHocPhan,
        hocPhan,
        thoiGian,
        diaDiem,
        siSo,
        daDk,
        soTc,
        hocPhi,
        ghiChu,
      ] = subjectHTML.map((sj) => sj.innerText.trim());

      // Lưu lại dữ liệu lịch học từng môn
      timeTable.push({
        STT,
        lopHocPhan,
        hocPhan,
        thoiGian,
        diaDiem,
        siSo,
        daDk,
        soTc,
        hocPhi,
        ghiChu,
      });
    }

    /* Chuyển dữ liệu thời gian từng môn học về dạng thời gian theo từng giai đoạn học  */
    timeTable.forEach((data, i) => {
      // Chuẩn hóa thời gian
      let getTimePhase = data.thoiGian.split("Từ");
      getTimePhase = getTimePhase.filter((time) => time != "");
      let phases = [];
      for (let time of getTimePhase) {
        let timePhase = time.split("\n");
        timePhase = timePhase.filter((time) => time !== "");
        let ranges = [];
        let [, start, end] = data_pattern
          .exec(timePhase.shift())
          .map((x) => x.trim());
        do {
          let [, day, periods, type] = time_pattern.exec(timePhase.shift());
          periods = periods.split(",");
          ranges.push({ day, periods, type });
        } while (timePhase.length > 0);
        phases.push({ start, end, ranges, location: "" });
      }

      // Chuẩn hóa địa chỉ học
      if (phases.length > 1) {
        let matches;
        do {
          let matches = location_pattern.exec(data.diaDiem);
          if (matches) {
            let [, phase, location] = matches;
            phases[+phase - 1].location = location;
          }
        } while (matches);
      } else {
        phases[0].location = data.diaDiem;
      }
      timeTable[i].thoiGian = phases;
      delete timeTable[i].diaDiem;
    });
```

Ấn chạy thử nào 😅 mong nó sẽ ra kết quả mong muốn

![Lich hoc](img/student-api/lich-hoc.png)

**Dữ liệu từng môn sẽ có dạng như sau:**

```
{
  STT: "1",
  lopHocPhan: "Bóng chuyền 1-2-19 (60.N01)",
  hocPhan: "TDUC141",
  thoiGian: [
    {
      start: "13/04/2020",
      end: "07/06/2020",
      ranges: [{ day: "3", periods: ["7", "8", "9"], type: "LT" }],
      location: "Sân BC 1 GDTC",
    },
  ],
  siSo: "45",
  daDk: "55",
  soTc: "1",
  hocPhi: "305.000",
  ghiChu: "",
}
```

Well done lại chạy ngon thế mới lạ :v, giờ ta sẽ sử dụng API **_evaluate_** cuả puppeteer để chạy như console log nhé

Lưu ý : API này sẽ trả về một Promise nhé

Tạo một file _StudentTimeTable.js_ trong page:

```
const {
  termSelector,
  tableSelector,
} = require("../selectors/StudentTimeTable");

module.exports = async (browser, term) => {
  const page = await browser.newPage();
  await page.goto(
    "http://dkh.tlu.edu.vn/CMCSoft.IU.Web.Info/Reports/Form/StudentTimeTable.aspx"
  );
  await page.type(termSelector, term);
  await page.waitForSelector(tableSelector);
  const timeTable = await page.evaluate(() => {
    *** NHÉT CÁI CODE VỪA CHẠY Ở CONSOLE VÀO ĐÂY ***
    return timeTable;
  });
  await page.close();
  return timeTable;
};

```

Vào _index.js_ thêm cái hàm này vào Class chính thôi nào :

```
...
  async StudentTimeTable(term) {
    const browser = await this.Browser();
    if (!this._timeTable) {
      try {
        this._timeTable = await StudentTimeTable(browser, term);
      } catch (error) {
        throw new Error("Không lấy được dữ liệu môn học");
      }
    }
    return this._timeTable;
  }
...
```

**Quay vô test thêm vào index.js chạy thử thui :**

```
test("getStudentTimeTable", async (t) => {
  const api = new StudentAPI();
  const user = { idUser: "Tài khoản", passwordUser: "Mật khẩu" };
  await api.Login(user);
  const lich = await api.StudentTimeTable("2_2019_2020");
  t.is(api.isAuthenticated, true);
  t.is(api.user, user);
  t.is(!!lich.length, true);
});

```

Kết quả thu được :

```
3 tests passed
```

TADAAAAAA 🥳🥳🥳 Chúng ta đã lấy xong dữ liệu về môn học rồi chuyển nó sang dạng timeline thôi nào .

## Phần này sẽ là phần **Challenge** của các bạn nha code mẫu của mình bên dưới, bạn bạn có thể sử lý theo nhiều các khác nhau nhé.

```
const moment = require("moment-timezone");
const _ = require("lodash");
const periodBoard = require("./periodBoard");
moment.tz.setDefault("Asia/Ho_Chi_Minh");
moment.locale("vi-VN");

const parseTime = (time) => moment(time, "DD/MM/YYYY");

const generateTimestamps = (start, end, day) => {
  let timeLine = [];
  // Set ngày bắt đầu
  start.weekday(day);
  // Kiểm tra ngày bắt đầu còn nhỏ hơn ngày kết thúc không , nếu có thêm vào timeline
  while (start.isSameOrBefore(end)) {
    timeLine.push(start.clone());
    //Tăng thêm 1 tuần để kiểm tra tiếp
    start.add(1, "week");
  }
  return timeLine;
};

//Chuyển thời gian sang thời gian học và kết thúc từng môn
const generateClasses = (timeRanges, startPeriod, endPeriod) => {
  return timeRanges.map((timeStamps) => ({
    start: timeStamps
      .clone()
      .hour(periodBoard[startPeriod].start.hour)
      .minute(periodBoard[startPeriod].start.minute),
    end: timeStamps
      .clone()
      .hour(periodBoard[endPeriod].end.hour)
      .minute(periodBoard[endPeriod].end.minute),
  }));
};

// Tạo time line
const generateTimeline = (subjects) => {
  let timelines = [];
  for (let subject of subjects) {
    for (let phase of subject.thoiGian) {
      for (let range of phase.ranges) {
        console.log(range);
        let timeline = generateClasses(
          generateTimestamps(
            parseTime(phase.start),
            parseTime(phase.end),
            parseInt(range.day) - 2
          ),
          range.periods[0],
          range.periods[range.periods.length - 1]
        );
        timeline.forEach((timestamp) => {
          let data = { timestamp, ...subject, ...range };
          delete data.thoiGian;
          delete data.STT;
          timelines.push(data);
        });
      }
    }
  }
  timelines.sort((first, last) => first.timestamp.start - last.timestamp.start);
  return timelines;
};
//
const generateTimelineByDay = (timelines) => {
  let days = _.groupBy(timelines, (timeline) => {
    return timeline.timestamp.start.clone().startOf("day");
  });
  return days;
};

module.exports = { generateTimeline, generateTimelineByDay };


```

Kết quả thu được sẽ dạng như này theo cách mình phân tích

```
{
  "Thu Jul 30 2020 00:00:00 GMT+0700": [
  {
    timestamp: {
      start: Moment<2020-07-30T07:00:00+07:00>,
      end: Moment<2020-07-30T08:45:00+07:00>
    },
    lopHocPhan: 'Lập trình nâng cao-2-19 (60TH4)',
    hocPhan: 'CSE381',
    siSo: '74',
    daDk: '73',
    soTc: '3',
    hocPhi: '915.000',
    ghiChu: '',
    day: '5',
    periods: [ '1', '2' ],
    type: 'LT'
  }
  ],
  ...
}
```

## Vậy là kết thúc **_Section 1 : "Làm thế nào để lấy dữ liệu từ web trường"_** rồi đó mong các bạn sẽ biết thêm nhiều điều. Có gì sai sót các bạn có thể đóng góp ý kiến cho mình nhé để mình cải thiện hơn ở Section 2

### **GitHub Repo** : https://github.com/2ksoft/student-api

### **NPM package** : https://www.npmjs.com/package/student-api

Ngoài ra các bạn có thể tham khảo thêm tại : https://github.com/t-rekttt/tinchi-api
