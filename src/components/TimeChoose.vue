<template>
  <div id="app">
    <div class="mt40">
      <div>With Time Picker</div>
      <div class="c-datepicker-date-editor c-datepicker-single-editor J-datepicker mt10">
        <i class="c-datepicker-range__icon kxiconfont icon-clock"></i>
        <input type="text" autocomplete="off" name="" placeholder="Select" class="c-datepicker-data-input">
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import moment from 'moment'
const DATEPICKERAPI = {
  initShowObject: function (_this, dataFormat) {
    let year, month, dayYear, dayMonth, dayDate;
    if (_this.config.isRange) {
      // 默认值回填时间插件选择框
      _this.fillDefault();
      dayYear = [dataFormat[0].year, dataFormat[1].year];
      dayMonth = [dataFormat[0].month, dataFormat[1].month];
      dayDate = [dataFormat[0].day, dataFormat[1].day];
      year = dataFormat[0].year;
      month = dataFormat[0].month;
    } else {
      const inputVal = _this.$input.value;
      year = dataFormat.year;
      month = dataFormat.month;
      dayYear = year;
      dayMonth = month;
      dayDate = inputVal? dataFormat.day : false;
      // 年
      if (_this.params.format[0]) {
        _this.yearObject = new Year(_this);
        if (!_this.params.format[2] &&!_this.params.format[1]) {
          _this.yearObject.render(year);
        }
      }
      // 月
      if (_this.params.format[1]) {
        _this.monthObject = new Month(_this);
        if (!_this.params.format[2]) {
          _this.$container.querySelectorAll('.c-datepicker-date-picker__prev-btn.month,.c-datepicker-date-picker__next-btn.month').forEach(btn => btn.style.display = 'none');
          _this.monthObject.render(month);
        }
      }
    }
    // 日
    if (_this.params.format[2]) {
      _this.dayObject = new Day(_this);
      _this.dayObject.render(dayYear, dayMonth, dayDate);
    }
    if (_this.params.format[3] || _this.params.format[4] || _this.params.format[5]) {
      _this.timeObject = new Time(_this);
    }
  },
  initParams: function (_this) {
    _this.splitStr = _this.config.format.replace(/[YMDhms:\s]/g, '').split('')[0];
    _this.params.format = API.getFormat(_this.config.format);
    // 最大值最小值判断
    _this.minJson = _this.config.min? API.getTimeFormat(moment(API.newDateFixed(_this, _this.config.min))) : false;
    _this.maxJson = _this.config.max? API.getTimeFormat(moment(API.newDateFixed(_this, _this.config.max))) : false;
  },
  renderPicker: function (target, isBlur) {
    const _this = API.getPicker(target);
    if (_this.config.isRange) {
      DATEPICKERAPI.renderPickerRange(target, isBlur);
    } else {
      DATEPICKERAPI.renderPickerSingle(target, isBlur);
    }
  },
  renderPickerRange: function (target, isBlur) {
    const _this = API.getPicker(target);
    let val = target.value;
    const format = _this.config.format.split(' ')[0];
    const regText = format.replace(/YYYY/, '[0-9]{4}').replace(/(MM|DD)/g, '[0-9]{2}');
    const reg = new RegExp('^' + regText + '$');
    // 判断符合时间格式
    if (reg.test(val)) {
      const $days = _this.$container.querySelectorAll('.c-datePicker__input-day');
      const $times = _this.$container.querySelectorAll('.c-datePicker__input-time');
      const index = Array.from($days).indexOf(target);
      const isBaseEnd = index === 1;
      const anotherVal = $days[index === 0? 1 : 0].value;
      const _moment = moment(API.newDateFixed(_this, val));
      const _momentAnother = moment(API.newDateFixed(_this, anotherVal));
      const orderFail = index === 0? _moment.isAfter(_momentAnother) : _moment.isBefore(_momentAnother);
      // 反过来，需要交换
      if (orderFail) {
        const temp = val;
        val = anotherVal;
        anotherVal = temp;
        _moment = moment(API.newDateFixed(_this, val));
        _momentAnother = moment(API.newDateFixed(_this, anotherVal));
        $days[index].value = val;
        $days[index === 0? 1 : 0].value = anotherVal;
      }
      // 十分秒重置
      if (_this.hasTime &&!isBlur) {
        $times[0].value = _this.timeMin;
        $times[1].value = _this.timeMax;
      }
      const resultAnother = API.getTimeFormat(_momentAnother);
      const result = API.getTimeFormat(_moment);
      // 最大最小值判断修改
      const resultJson = API.minMaxFill(_this, result, index);
      result = resultJson.result;
      // 填充值
      target.value = resultJson.val;
      const rangeYears = [], rangeMonths = [], rangeDates = [];
      rangeYears[index] = result.year;
      rangeMonths[index] = result.month;
      rangeDates[index] = result.day;
      rangeYears[1 - index] = resultAnother.year;
      rangeMonths[1 - index] = resultAnother.month;
      rangeDates[1 - index] = resultAnother.day;
      _this.dayObject.renderRange(rangeYears, rangeMonths, rangeDates, isBaseEnd, true);
    }
  },
  renderPickerSingle: function (target) {
    const _this = API.getPicker(target);
    let val = target.value;
    const format = _this.config.format.split(' ')[0];
    const regText = format.replace(/YYYY/, '[0-9]{4}').replace(/(MM|DD)/g, '[0-9]{2}');
    const reg = new RegExp('^' + regText + '$');
    // 判断符合时间格式
    if (reg.test(val)) {
      const $time = _this.$container.querySelectorAll('.c-datePicker__input-time');
      const _moment = moment(API.newDateFixed(_this, val));
      const result = API.getTimeFormat(_moment);
      // 最大最小值判断修改
      const resultJson = API.minMaxFill(_this, result, 0);
      result = resultJson.result;
      val = resultJson.val;
      // 填充值
      target.value = val;
      if (_this.hasTime) {
        val += ' ' + $time.value;
      }
      _this.$input.value = val;
      _this.dayObject.renderSingle(result.year, result.month, result.day, true);
    }
  },
  cancelBlur: function (_this) {
    // $.unsub('datapickerRenderPicker');
    _this.isBlur = false;
  },
  renderTimePanelHtml: function (_this, type, hour, minute, second) {
    hour = hour || moment().hours();
    minute = minute || moment().minutes();
    second = second || moment().seconds();
    let li = '';
    let html = '';
    // 时
    if (type[0]) {
      for (let i = 0; i < 24; i++) {
        const className = hour === i? 'active' : '';
        li += RENDERAPI.timeLiTpl(className, API.fillTime(i));
      }
      html += RENDERAPI.timeTpl('hour', li);
      li = '';
    }
    // 分
    if (type[1]) {
      for (let j = 0; j < 60; j++) {
        const className = minute === j? 'active' : '';
        li += RENDERAPI.timeLiTpl(className, API.fillTime(j));
      }
      html += RENDERAPI.timeTpl('minute', li);
      li = '';
    }
    // 秒
    if (type[2]) {
      for (let k = 0; k < 60; k++) {
        const className = second === k? 'active' : '';
        li += RENDERAPI.timeLiTpl(className, API.fillTime(k));
      }
      html += RENDERAPI.timeTpl('second', li);
    }
    const nameOptions = $.fn.datePicker.dates[_this.language];
    html = RENDERAPI.timeMainTpl(nameOptions, html);
    return html;
  },
  setInitVal: function (_this) {
    _this.params.initBeginVal = _this.$inputBegin.value;
    _this.params.initEndVal = _this.$inputEnd.value;
  }
};
onMounted(() => {
  document.body.addEventListener('click.datePicker', () => {
    document.querySelectorAll('.c-datepicker-picker').forEach((panel) => {
      const _this = panel.dataset.picker;
      if (panel.style.display === 'block') {
        if (_this.config.isRange && (!_this.$inputBegin.value &&!_this.$inputEnd.value)) {
          panel.querySelectorAll('td.available').forEach((td) => td.classList.remove('current', 'in-range'));
        }
        if (_this.hasTime) {
          panel.querySelectorAll('.c-datepicker-time-panel').forEach((timePanel) => (timePanel.style.display = 'none'));
        }
        if (_this.onlyTime) {
          _this.datePickerObject.fixedInputValOnlyTime();
        } else {
          _this.datePickerObject.fixedInputVal();
        }
        _this.$container.dataset.isShow = false;
        _this.config.hide.call(_this, 'clickBody');
        _this.datePickerObject.betweenHandle();
      }
    });
    document.querySelectorAll('.c-datepicker-picker').forEach((panel) => (panel.style.display = 'none'));
  });
});
onMounted(() => {
  document.querySelectorAll('.c-datepicker-box').forEach((box) => {
    box.addEventListener('scroll', scrollSetContainerPos);
  });
});

function scrollSetContainerPos() {
  document.querySelectorAll('.c-datepicker-picker').forEach((panel) => {
    const _this = panel.dataset.picker;
    if (panel.style.display === 'block') {
      setContainerPos(_this.datePickerObject);
    }
  });
}
class SingleDatePicker {
  constructor(datePickerObject) {
    this.datePickerObject = datePickerObject;
    this.datePickerObject.pickerObject = null;
    this.$input = datePickerObject.$target.querySelector('input');
    this.config = datePickerObject.config;
    this.params = {};
    this.language = this.config.language || 'zh-CN';
    this.hasTime = this.config.format.split(' ').length > 1;
    if (this.hasTime) {
      this.timeMin = API.timeVal(this, 'min');
      this.timeMax = API.timeVal(this, 'max');
    }
    this.init();
  }

  init() {
    this.initShow();
    this.event();
  }

  initShow() {
    // 初始化 splitStr，params.format，minJson，maxJson
    DATEPICKERAPI.initParams(this);
    this.params.isYear = this.params.format[0] &&!this.params.format[1];
    this.params.isMonth = this.params.format[0] && this.params.format[1] &&!this.params.format[2];
    const table = '';
    const inputVal = this.$input.value;
    const result = inputVal? moment(API.newDateFixed(this, inputVal)) : moment();
    const dataFormat = API.getTimeFormat(result);
    let sidebar = '';
    let hasSidebar = '';
    let hasTime = '';
    if (this.params.format[3] || this.params.format[4] || this.params.format[5]) {
      hasTime = 'has-time';
    }
    // 有快捷键-单个还是范围
    if (this.config.hasShortcut) {
      hasSidebar = 'has-sidebar';
      sidebar = rederSidebar(this);
    }
    const nameOptions = $.fn.datePicker.dates[this.language];
    let renderTpl = RENDERAPI.datePickerMainTpl(nameOptions);
    if (this.params.isYear || this.params.isMonth) {
      renderTpl = renderTpl.replace(/{{footerButton}}/g, RENDERAPI.pickerFooterClearButton(nameOptions));
    } else {
      renderTpl = renderTpl.replace(/{{footerButton}}/g, RENDERAPI.pickerFooterNowButton(nameOptions));
    }
    const $datePickerHtml = $(renderTpl.replace(/{{table}}/g, table).replace(/{{year}}/g, dataFormat.year).replace(/{{month}}/g, dataFormat.month).replace('{{sidebar}}', sidebar).replace('{{hasTime}}', hasTime).replace('{{hasSidebar}}', hasSidebar));
    document.body.appendChild($datePickerHtml[0]);
    this.$container = $datePickerHtml;
    this.$container.dataset.picker = this;
    this.$container.classList.add('is-' + this.language);
    // 没有十分秒
    if (!this.hasTime) {
      this.$container.querySelector('.c-datepicker-date-picker__time-header').style.display = 'none';
    }
    // 初始化年月日十分秒 panel
    DATEPICKERAPI.initShowObject(this, dataFormat);
    // 默认值回填时间插件选择框
    const val = this.$input.value.split(' ');
    this.$container.querySelector('.c-datePicker__input-day').value = val[0];
    this.$container.querySelector('.c-datePicker__input-time').value = val[1];
    if (getMomentWhenEmpty(this).type!== 'active') {
      this.$container.querySelector('.c-datepicker-picker__btn-now').remove();
    }
  }

  event() {
    if (this.hasTime) {
      this.eventHasTime();
    }
    this.datePickerObject.$target.addEventListener('click', (event) => {
      event.stopPropagation();
    });
    this.$container.addEventListener('click', (event) => {
      event.stopPropagation();
    });

    // 点击选择年
    this.$container.addEventListener('click', '.c-datepicker-date-picker__header-year', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      if (_this.isBlur) {
        DATEPICKERAPI.cancelBlur(_this);
      }
      if (event.target.classList.contains('disabled')) {
        return;
      }

      let val = _this.$input.value;
      if (!val) {
        val = moment();
      } else {
        val = moment(API.newDateFixed(_this, val));
      }
      _this.yearObject.render(val.year());
    });

    // 点击选择月
    this.$container.addEventListener('click', '.c-datepicker-date-picker__header-month', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      if (_this.isBlur) {
        DATEPICKERAPI.cancelBlur(_this);
      }
      if (event.target.classList.contains('disabled')) {
        return;
      }
      let val = _this.$input.value;
      if (!val) {
        val = moment();
      } else {
        val = moment(API.newDateFixed(_this, val));
      }
      _this.monthObject.render(val.month() + 1);
    });

    // 下一月
    this.$container.addEventListener('click', '.c-datepicker-date-picker__next-btn.month', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      renderYearMonth(_this, 'next', 'month');
    });

    // 上一月
    this.$container.addEventListener('click', '.c-datepicker-date-picker__prev-btn.month', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      // 操作选择年的翻页
      renderYearMonth(_this, 'prev', 'month');
    });

    // 下一年
    this.$container.addEventListener('click', '.c-datepicker-date-picker__next-btn.year', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      // 操作选择年的翻页
      if (event.target.classList.contains('is-year')) {
        const newYear = Number(_this.$container.querySelector('.c-datepicker-year-table td.available').firstElementChild.querySelector('.cell').textContent) + 10;
        _this.yearObject.render(newYear);
      } else if (event.target.classList.contains('is-month')) {
        const $year = _this.$container.querySelector('.c-datepicker-date-picker__header-year span');
        $year.textContent = Number($year.textContent) + 1;
        _this.monthObject.render();
      } else {
        renderYearMonth(_this, 'next', 'year');
      }
    });

    // 上一年
    this.$container.addEventListener('click', '.c-datepicker-date-picker__prev-btn.year', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      // 操作选择年的翻页
      if (event.target.classList.contains('is-year')) {
        const newYear = Number(_this.$container.querySelector('.c-datepicker-year-table td.available').firstElementChild.querySelector('.cell').textContent) - 10;
        _this.yearObject.render(newYear);
      } else if (event.target.classList.contains('is-month')) {
        const $year = _this.$container.querySelector('.c-datepicker-date-picker__header-year span');
        $year.textContent = Number($year.textContent) - 1;
        _this.monthObject.render();
      } else {
        renderYearMonth(_this, 'prev', 'year');
      }
    });

    function renderYearMonth(_this, dire, type) {
      if (_this.isBlur) {
        _this.dayObject.prevNextSingle(dire, type);
        // $.unsub('datapickerRenderPicker');
        _this.isBlur = false;
        // $.sub('datapickerClick', function (e) {
        //   _this.dayObject.prevNextSingle(dire, type);
        //   $.unsub('datapickerClick');
        // });
      } else {
        _this.dayObject.prevNextSingle(dire, type);
      }
    }

    // 点击此刻
    this.$container.addEventListener('click', '.c-datepicker-picker__btn-now', () => {
      const _this = API.getPicker(event.target);
      setValue(_this, moment().format(_this.config.format));
      _this.datePickerObject.hide('shortcut');
    });

    // 点击清空
    this.$container.addEventListener('click', '.c-datepicker-picker__btn-clear', () => {
      const _this = API.getPicker(event.target);
      _this.clear();
    });

    // 点击快捷选项
    this.$container.addEventListener('click', '.c-datepicker-picker__shortcut', () => {
      const _this = API.getPicker(event.target);
      const day = event.target.dataset.value;
      let result = moment().add(day, 'day').format(_this.config.format);
      if (_this.hasTime) {
        const time = event.target.dataset.time;
        if (time) {
          result = result.split(' ')[0] + ' ' + time;
        }
      }
      setValue(_this, result);
      _this.datePickerObject.hide('shortcut');
    });

    // 点击确定
    this.$container.addEventListener('click', '.c-datepicker-picker__link-btn.confirm', () => {
      const _this = API.getPicker(event.target);
      if (!_this.$input.value) {
        const _moment = getMomentWhenEmpty(_this).value;
        setValue(_this, _moment);
      }
      _this.datePickerObject.hide('confirm');
    });
  }

  eventHasTime() {
    // 输入框修改日期 input
    this.$container.addEventListener('keyup', '.c-datePicker__input-time', (event) => {
      const _this = API.getPicker(event.target);
      // 更新显示的 time panel 值
      const isMatch = _this.timeObject.updateTimePanel();
      if (isMatch) {
        const time = event.target.value;
        const day = _this.$container.querySelector('.c-datePicker__input-day').value;
        _this.$input.value = day + ' ' + time;
      }
    });
    this.$container.addEventListener('click', '.c-datePicker__input-time', (event) => {
      event.stopPropagation();
    });
    // 修改日期
    this.$container.addEventListener('keyup', '.c-datePicker__input-day', () => {
      DATEPICKERAPI.renderPickerSingle(event.target);
    });
    // 失焦判断最大值最小值
    this.$container.addEventListener('blur', '.c-datePicker__input-day', (event) => {
      const _this = API.getPicker(event.target);
      // 修复满足格式但不完全符合的 day 格式修正
      fillDay(_this, event.target);
      API.judgeTimeRange(_this, event.target, _this.$container.querySelector('.c-datePicker__input-time'));
    });
    // 失焦判断最大值最小值
    this.$container.addEventListener('blur', '.c-datePicker__input-time', (event) => {
      const _this = API.getPicker(event.target);
      // 修复满足格式但不完全符合的 time 格式修正
      fillTime(_this, event.target);
      API.judgeTimeRange(_this, _this.$container.querySelector('.c-datePicker__input-day'), event.target);
    });

    // 聚焦时分秒 input
    this.$container.addEventListener('focus', '.c-datePicker__input-time', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      if (!_this.$input.value &&!event.target.value) {
        let now = moment().format(_this.config.format);
        _this.$input.value = now;
        now = now.split(' ');
        _this.$container.querySelector('.c-datePicker__input-day').value = now[0];
        event.target.value = now[1];
      }
      _this.activeTimeWrap = event.target.closest('.c-datepicker-date-picker__time-header');
      const val = event.target.value.split(':');
      _this.showTimeSelect(val[0], val[1], val[2]);
    });
    // 聚焦日期 input
    this.$container.addEventListener('focus', '.c-datePicker__input-day', () => {
      const _this = API.getPicker(event.target);
      if (!_this.$input.value) {
        const now = moment().format(_this.config.format).split(' ');
        event.target.value = now[0];
        if (now.length > 1) {
          _this.$container.querySelector('.c-datePicker__input-time').value = now[1];
        }
      }
    });
  }

  clear() {
    this.$input.value = '';
    this.$container.querySelectorAll('td.available').forEach((td) => td.classList.remove('current'));
  }

  show() {
    // 日
    if (this.params.format[2]) {
      const val = API.getRangeTimeFormat(this, this.$input);
      this.dayObject.render(val.year, val.month, val.day, true);
    }
    this.$container.style.display = 'block';
  }

  reRenderDay() {
    // 日
    if (this.params.format[2]) {
      const result = API.getRangeTimeFormat(this, this.$input);
      // 判断是否选中
      const _val = this.$input.value? result.day : false;
      this.dayObject.render(result.year, result.month, _val, true);
    }
  }

  renderYear() {
    this.yearObject.render();
  }

  renderMonth() {
    this.monthObject.render();
  }

  showTimeSelect(year, month, day) {
    if (this.params.format[3] || this.params.format[4] || this.params.format[5]) {
      this.timeObject.render(this.params.format.slice(3), year, month, day);
    }
  }
}
class RangeDatePicker {
  constructor(datePickerObject) {
    this.datePickerObject = datePickerObject;
    this.datePickerObject.pickerObject = null;
    this.$input = datePickerObject.$target.querySelectorAll('input');
    this.$inputBegin = this.$input[0];
    this.$inputEnd = this.$input[1];
    this.config = datePickerObject.config;
    this.params = {};
    this.language = this.config.language || 'zh-CN';
    this.hasTime = this.config.format.split(' ').length > 1;
    if (this.hasTime) {
      this.timeMin = API.timeVal(this, 'min');
      this.timeMax = API.timeVal(this, 'max');
    }
    this.init();
  }

  init() {
    this.initShow();
    this.event();
  }

  initShow() {
    // 初始化 splitStr，params.format，minJson，maxJson
    DATEPICKERAPI.initParams(this);
    const table = '';
    const dataFormat = [];
    dataFormat[0] = API.getRangeTimeFormat(this, this.$inputBegin);
    dataFormat[1] = API.getRangeTimeFormat(this, this.$inputEnd);
    let sidebar = '';
    let hasSidebar = '';
    let hasTime = '';
    if (this.params.format[3] || this.params.format[4] || this.params.format[5]) {
      hasTime = 'has-time';
    }
    // 有快捷键-单个还是范围
    if (this.config.hasShortcut) {
      hasSidebar = 'has-sidebar';
      sidebar = rederSidebar(this);
    }
    const nameOptions = $.fn.datePicker.dates[this.language];
    const $datePickerHtml = $(RENDERAPI.rangePickerMainTpl(nameOptions, hasTime, hasSidebar, dataFormat[1].year, dataFormat[1].month, sidebar, table));
    document.body.appendChild($datePickerHtml[0]);
    this.$container = $datePickerHtml;
    this.$container.dataset.picker = this;
    this.$container.classList.add('is-' + this.language);
    // 没有十分秒
    if (!this.hasTime) {
      this.$container.querySelector('.c-datepicker-date-range-picker__time-header').style.display = 'none';
    }
    // 初始化年月日十分秒 panel
    DATEPICKERAPI.initShowObject(this, dataFormat);
  }

  // 默认值回填时间插件选择框
  fillDefault() {
    const valBegin = this.$inputBegin.value.split(' ');
    const valEnd = this.$inputEnd.value.split(' ');
    const $day = this.$container.querySelectorAll('.c-datePicker__input-day');
    const $time = this.$container.querySelectorAll('.c-datePicker__input-time');
    if (valBegin) {
      $day[0].value = valBegin[0];
      $time[0].value = valBegin[1];
    }
    if (valEnd) {
      $day[1].value = valEnd[0];
      $time[1].value = valEnd[1];
    }
  }

  event() {
    if (this.hasTime) {
      this.eventHasTime();
    }
    this.$container.addEventListener('click', (event) => {
      event.stopPropagation();
    });
    this.datePickerObject.$target.addEventListener('click', (event) => {
      event.stopPropagation();
    });

    // 下一月
    this.$container.addEventListener('click', '.c-datepicker-date-range-picker__next-btn.month', () => {
      const _this = API.getPicker(event.target);
      renderYearMonth(_this, 'next', 'month');
    });
    // 上一月
    this.$container.addEventListener('click', '.c-datepicker-date-range-picker__prev-btn.month', () => {
      const _this = API.getPicker(event.target);
      renderYearMonth(_this, 'prev', 'month');
    });
    // 下一年
    this.$container.addEventListener('click', '.c-datepicker-date-range-picker__next-btn.year', () => {
      const _this = API.getPicker(event.target);
      renderYearMonth(_this, 'next', 'year');
    });
    // 上一年
    this.$container.addEventListener('click', '.c-datepicker-date-range-picker__prev-btn.year', () => {
      const _this = API.getPicker(event.target);
      renderYearMonth(_this, 'prev', 'year');
    });

    function renderYearMonth(_this, dire, type) {
      if (_this.isBlur) {
        // $.sub('datapickerClick', function (e) {
        //   _this.dayObject.prevNextRender(dire, type);
        //   $.unsub('datapickerClick');
        // });
        // $.pub('datapickerRenderPicker');
        _this.dayObject.prevNextRender(dire, type);
      } else {
        _this.dayObject.prevNextRender(dire, type);
      }
    }
    // 点击清空
    this.$container.addEventListener('click', '.c-datepicker-picker__btn-clear', () => {
      const _this = API.getPicker(event.target);
      _this.clear();
    });

    // 快捷选项
    this.$container.addEventListener('click', '.c-datepicker-picker__shortcut', () => {
      const _this = API.getPicker(event.target);
      const days = event.target.dataset.value.split(',');
      let begin = moment().add(days[0], 'day').format(_this.config.format);
      let end = moment().add(days[1], 'day').format(_this.config.format);
      if (_this.hasTime) {
        const times = event.target.dataset.time.split(',');
        if (times[0]) {
          begin = begin.split(' ')[0] + ' ' + times[0];
        }
        if (times[1]) {
          end = end.split(' ')[0] + ' ' + times[1];
        }
      }
      _this.$inputBegin.value = begin;
      _this.$inputEnd.value = end;
      _this.datePickerObject.hide('shortcut');
      // setValue(_this, result);
    });

    // 点击确定
    this.$container.addEventListener('click', '.c-datepicker-picker__link-btn.confirm', () => {
      const _this = API.getPicker(event.target);
      const $days = _this.$container.querySelectorAll('.c-datePicker__input-day');
      const $times = _this.$container.querySelectorAll('.c-datePicker__input-time');
      let start = $days[0].value;
      let end = $days[1].value;
      if (!start ||!end) {
        _this.datePickerObject.hide('confirm');
        return;
      }
      if (_this.hasTime) {
        start += ' ' + $times[0].value;
        end += ' ' + $times[1].value;
      }
      _this.$inputBegin.value = start;
      _this.$inputEnd.value = end;
      _this.datePickerObject.hide('confirm');
    });
  }

  eventHasTime() {
    // 输入框修改日期 input
    this.$container.addEventListener('keyup', '.c-datePicker__input-time', (event) => {
      const _this = API.getPicker(event.target);
      _this.timeObject.updateTimePanel();
    });
    // 输入框修改日期 input
    this.$container.addEventListener('keyup', '.c-datePicker__input-day', () => {
      DATEPICKERAPI.renderPicker(this);
    });

    this.$container.addEventListener('click', '.c-datePicker__input-time', (event) => {
      event.stopPropagation();
    });
    // 聚焦时分秒 input
    this.$container.addEventListener('focus', '.c-datePicker__input-time', (event) => {
      event.stopPropagation();
      const _this = API.getPicker(event.target);
      if (!_this.$input.value &&!event.target.value) {
        let now = moment().format(_this.config.format);
        // _this.$input.val(now);
        now = now.split(' ');
        _this.$container.querySelectorAll('.c-datePicker__input-day')[0].value = now[0];
        _this.$container.querySelectorAll('.c-datePicker__input-time')[0].value = now[1];
      }
      _this.activeTimeWrap = event.target.closest('.c-datepicker-date-range-picker__time-content');
      _this.showTimeSelect();
      event.target.dispatchEvent(new Event('keyup'));
    });
    // 聚焦日期 input
    this.$container.addEventListener('focus', '.c-datePicker__input-day,.c-datePicker__input-time', (event) => {
      const _this = API.getPicker(event.target);
      const $day = _this.$container.querySelectorAll('.c-datePicker__input-day');
      if (!event.target.value) {
        const now = moment().format(_this.config.format).split(' ');
        $day[0].value = now[0];
        if (now.length > 1) {
          _this.$container.querySelectorAll('.c-datePicker__input-time')[0].value = now[1];
        }
      }
    });
    // day 失焦判断最大值最小值
    this.$container.addEventListener('blur', '.c-datePicker__input-day', (event) => {
      const _this = API.getPicker(event.target);
      const index = Array.from(_this.$container.querySelectorAll('.c-datePicker__input-day')).indexOf(event.target);
      // 修复满足格式但不完全符合的 day 格式修正
      fillDay(_this, event.target);
      API.judgeTimeRange(_this, event.target, _this.$container.querySelectorAll('.c-datePicker__input-time')[index], index);
    });
    // time 失焦判断最大值最小值
    this.$container.addEventListener('blur', '.c-datePicker__input-time', (event) => {
      const _this = API.getPicker(event.target);
      const index = Array.from(_this.$container.querySelectorAll('.c-datePicker__input-time')).indexOf(event.target);
      // 修复满足格式但不完全符合的 time 格式修正（先修正，后比较大小）
      fillTime(_this, event.target);
      API.judgeTimeRange(_this, _this.$container.querySelectorAll('.c-datePicker__input-day')[index], event.target, index);
    });
  }

  show() {
    this.fillDefault();
    const dataFormat = [];
    dataFormat[0] = API.getRangeTimeFormat(this, this.$inputBegin);
    dataFormat[1] = API.getRangeTimeFormat(this, this.$inputEnd);
    const yearArr = [dataFormat[0].year, dataFormat[1].year];
    const monthArr = [dataFormat[0].month, dataFormat[1].month];
    const dayArr = [dataFormat[0].day, dataFormat[1].day];
    // 日
    if (this.params.format[2]) {
      this.dayObject.render(yearArr, monthArr, dayArr, false, true);
    }
    DATEPICKERAPI.setInitVal(this);
    this.$container.style.display = 'block';
  }

  clear() {
    this.$inputBegin.value = '';
    this.$inputEnd.value = '';
    const $inputs = this.$container.querySelectorAll('.c-datePicker__input-day,.c-datePicker__input-time');
    $inputs.forEach(input => input.value = '');
    this.$container.querySelectorAll('td.available').forEach(td => td.classList.remove('current', 'in-range'));
  }

  renderYear() {
    this.yearObject.render();
  }

  renderMonth() {
    this.monthObject.render();
  }

  showTimeSelect() {
    if (this.params.format[3] || this.params.format[4] || this.params.format[5]) {
      this.timeObject.render(this.params.format.slice(3));
    }
  }
}
class DatePicker {
  constructor(options, ele) {
    // this.$container = $('.c-datepicker-picker');
    const $target = ref(ele);
    const config = {...defaultOptions,...options };
    const params = {};
    // 只有时分秒，没有日期
    const onlyTime = API.onlytimeReg(config.format);
    this.init();
  }

  init() {
    if (!this.config.isRange) {
      this.pickerObject = this.onlyTime
       ? new RangeDatePickerTime(this)
        : new SingleDatePicker(this);
    } else {
      this.pickerObject = this.onlyTime
       ? new RangeDatePickerTime(this)
        : new RangeDatePicker(this);
    }
    this.pickerObject.$input.dataset.datepicker = this;
    this.event();
  }

  event() {
    this.pickerObject.$input.addEventListener('click', () => {
      const _this = this.pickerObject.$input.dataset.datepicker;
      if (!_this.pickerObject.$container.dataset.isShow) {
        // 重置状态
        document.querySelectorAll('.c-datepicker-picker').forEach(picker => picker.dataset.isShow = false);
        _this.pickerObject.$container.dataset.isShow = true;
        _this.show();
      }
    });

    this.pickerObject.$input.addEventListener('focus', () => {
      const _this = this.pickerObject.$input.dataset.datepicker;
      _this.initInputVal = this.pickerObject.$input.value;
    });

    // 兼容输入框失去焦点
    this.pickerObject.$container.addEventListener('click', () => {
      const _this = this.pickerObject.$container.dataset.picker;
      if (_this.isBlur) {
        // $.unsub('datapickerClick');
        // $.pub('datapickerRenderPicker');
        _this.isBlur = false;
      }
    });

    // 输入框失去焦点
    this.pickerObject.$input.addEventListener('blur', () => {
      if (!this.pickerObject.$input.value) {
        return;
      }
      const _this = this.pickerObject.$input.dataset.datepicker;
      const index = Array.from(this.pickerObject.$input).indexOf(this.pickerObject.$input);
      const valArr = this.pickerObject.$input.value.split(' ');
      let day = valArr[0];
      const $container = _this.pickerObject.$container;
      // 有十分秒
      if (_this.pickerObject.hasTime) {
        let time = _this.onlyTime
         ? API.timeCheck(valArr[0])
          : valArr[1]? API.timeCheck(valArr[1]) : false;
        const $time = $container.querySelectorAll('.c-datePicker__input-time');
        const timeResult = time && time.match(API.timeReg(_this));
        if (_this.onlyTime) {
          // 无日期只有时分秒
          if (!time ||!timeResult) {
            time = _this.initInputVal;
            this.pickerObject.$input.value = _this.initInputVal;
          } else {
            if (timeResult) {
              time = timeResult[5]
               ? timeResult[1] + ':' + API.fillTime(timeResult[3]) + ':' + API.fillTime(timeResult[5])
                : timeResult[1] + ':' + API.fillTime(timeResult[3]);
            }
            this.pickerObject.$input.value = time;
          }
        } else {
          const dayReg = API.dayReg(_this.pickerObject);
          const $day = $container.querySelectorAll('.c-datePicker__input-day');
          const dayResult = day.match(dayReg);
          if (!time ||!timeResult ||!dayResult) {
            day = _this.initInputVal.split(' ')[0];
            time = _this.initInputVal.split(' ')[1];
            this.pickerObject.$input.value = _this.initInputVal;
          } else {
            if (dayResult) {
              // 兼容201808变为2018-00-08的情况
              dayResult = API.fixedFill(dayResult);
              day = dayResult[1] + _this.pickerObject.splitStr + API.fillTime(dayResult[3]) + _this.pickerObject.splitStr + API.fillTime(dayResult[5]);
            }
            if (timeResult) {
              time = timeResult[5]
               ? timeResult[1] + ':' + API.fillTime(timeResult[3]) + ':' + API.fillTime(timeResult[5])
                : timeResult[1] + ':' + API.fillTime(timeResult[3]);
            }
            this.pickerObject.$input.value = day + ' ' + time;
            $time[index].value = time;
            $day[index].value = day;
            // 兼容失去焦点，点击选择日期
            _this.pickerObject.isBlur = true;
            // 逻辑：先blur-绑定重新渲染date panel事件-触发选择日期或年月切换等事件-发布重新渲染事件|取消发布事件
            // 处理的类型有，点击切换上下一年月，点击年月选择，选择日期，点击piker其他
            // $.sub('datapickerRenderPicker', function () {
            //   if (!_this.onlyTime) {
            //     DATEPICKERAPI.renderPicker($day.eq(index)[0], true);
            //   }
            //   _this.pickerObject.isBlur = false;
            //   $.pub('datapickerClick');
            //   $.unsub('datapickerRenderPicker');
            // });
          }
        }
      } else {
        // 没有十分秒
        // 年月格式
        if (_this.pickerObject.params.isMonth) {
          const _moment = moment(API.newDateFixed(_this.pickerObject, day + _this.pickerObject.splitStr + '01'));
          const result = API.getTimeFormat(_moment);
          const resultJson = API.minMaxFill(_this.pickerObject, result, 0, 'month');
          const val = resultJson.val;
          this.pickerObject.$input.value = val;
          // _this.hide();
        } else if (_this.pickerObject.params.isYear) {
          if (_this.config.min && day < _this.config.min) {
            day = _this.config.min;
          }
          if (_this.config.max && day > _this.config.max) {
            day = _this.config.max;
          }
          this.pickerObject.$input.value = day;
        } else {
          const dayReg = API.dayReg(_this.pickerObject);
          const dayResult = day.match(dayReg);
          if (!dayResult) {
            // 不匹配
            this.pickerObject.$input.value = _this.initInputVal;
          } else {
            // 匹配
            // 兼容201808变为2018-00-08的情况
            dayResult = API.fixedFill(dayResult);
            day = dayResult[1] + _this.pickerObject.splitStr + API.fillTime(dayResult[3]) + _this.pickerObject.splitStr + API.fillTime(dayResult[5]);
            this.pickerObject.$input.value = day;
          }
        }
      }
    });
  }

  show() {
    setContainerPos(this);
    document.querySelectorAll('.c-datepicker-picker').forEach(picker => picker.style.display = 'none');
    this.pickerObject.show();
    this.config.show.call(this.pickerObject);
  }

  hide(type) {
    // 判断输入框没有值
    this.pickerObject.$container.querySelectorAll('.td.available').forEach(td => td.classList.remove('current', 'in-range'));
    this.pickerObject.$container.querySelectorAll('.c-datepicker-time-panel').forEach(panel => panel.style.display = 'none');
    this.pickerObject.$container.style.display = 'none';
    this.betweenHandle();
    // 判断范围，最大值最小值
    if (this.onlyTime) {
      this.fixedInputValOnlyTime();
    } else {
      this.fixedInputVal();
    }
    this.pickerObject.$container.dataset.isShow = false;
    this.config.hide.call(this.pickerObject, type);
  }

  fixedInputVal() {
    const _config = this.config;
    const _this = this.pickerObject;
    // 最大值最小值
    const hasMin = _this.minJson? true : false;
    const hasMax = _this.maxJson? true : false;
    const getMoment = function (_this, val) {
      return moment(API.newDateFixed(_this, val));
    };
    const minMoment = hasMin? getMoment(_this, _config.min) : false;
    const maxMoment = hasMax? getMoment(_this, _config.max) : false;
    if (_config.isRange) {
      const valBegin = _this.$inputBegin.value;
      const valEnd = _this.$inputEnd.value;
      if (!valBegin &&!valEnd) {
        return;
      }
      const momentBegin = valBegin? getMoment(_this, valBegin) : false;
      const momentEnd = valEnd? getMoment(_this, valEnd) : false;
      // 开始>结束=>设置为没改变时的值
      if (valBegin && valEnd && momentBegin.isAfter(momentEnd)) {
        _this.$inputBegin.value = _this.params.initBeginVal;
        _this.$inputEnd.value = _this.params.initEndVal;
        return;
      }
      // 开始>结束不在范围内
      if (hasMin && valBegin && momentBegin.isBefore(minMoment)) {
        _this.$inputBegin.value = _config.min;
      }
      if (hasMax && valEnd && momentEnd.isAfter(maxMoment)) {
        _this.$inputEnd.value = _config.max;
      }
    } else {
      const val = _this.$input.value;
      if (!val) {
        return;
      }
      const momentBegin = val? getMoment(_this, val) : false;
      // 开始>结束不在范围内
      if (hasMin && momentBegin.isBefore(minMoment)) {
        _this.$input.value = _config.min;
      }
      if (hasMax && momentBegin.isAfter(maxMoment)) {
        _this.$input.value = _config.max;
      }
    }
  }

  fixedInputValOnlyTime() {
    const _config = this.config;
    const _this = this.pickerObject;
    // 最大值最小值
    if (_config.isRange) {
      const valBegin = _this.$inputBegin.value;
      const valEnd = _this.$inputEnd.value;
      if ((!valBegin &&!valEnd)) {
        return;
      }
      const valBeginArr = valBegin.split(':');
      const valEndArr = valEnd.split(':');
      const valSecondBegin = API.countSecond(valBeginArr);
      const valSecondEnd = API.countSecond(valEndArr);
      if (valSecondBegin > valSecondEnd) {
        _this.$inputBegin.value = _this.params.initBeginVal;
        _this.$inputEnd.value = _this.params.initEndVal;
        return;
      }
      // var val1 = TIMEONLYAPI.checkMinMaxGetVal(_this, valBeginArr);
      // var val2 = TIMEONLYAPI.checkMinMaxGetVal(_this, valEndArr);

      const minSecond = _this.configMinMax.minSecond;
      const maxSecond = _this.configMinMax.maxSecond;
      if (valSecondBegin < minSecond) {
        _this.$inputBegin.value = _this.configMinMax.minVal;
      }
      if (valSecondEnd > maxSecond) {
        _this.$inputEnd.value = _this.configMinMax.maxVal;
      }
    } else {
      const val = _this.$input.value;
      if (!_this.configMinMax ||!val) {
        return;
      }
      const valArr = val.split(':');
      // 检查最大最小对比当前值，获取值
      val = TIMEONLYAPI.checkMinMaxGetVal(_this, valArr);
      _this.$input.value = val;
    }
  }

  betweenHandle() {
    const _config = this.config;
    // 处理范围间距，检测开始结束间隔时间
    if (!_config.isRange ||!_config.between) {
      return false;
    }
    const start = this.pickerObject.$inputBegin.value;
    const end = this.pickerObject.$inputEnd.value;
    if (!start ||!end) {
      return false;
    }
    const beginMoment = moment(API.newDateFixed(this.pickerObject, start));
    const endMoment = moment(API.newDateFixed(this.pickerObject, end));
    const beginFormat = API.getTimeFormat(beginMoment);
    const endFormat = API.getTimeFormat(endMoment);
    // 同一个月内
    if (_config.between === 'month') {
      if (beginFormat.year!== endFormat.year || beginFormat.month!== endFormat.month) {
        const val = beginMoment.set({ 'year': endFormat.year, 'month': endFormat.month - 1, 'date': 1 }).format(_config.format);
        this.pickerObject.$inputBegin.value = val;
      }
      return;
    }
    // 同一年内
    if (_config.between === 'year') {
      if (beginFormat.year!== endFormat.year) {
        const val = beginMoment.set({ 'year': endFormat.year, 'month': 0, 'date': 1 }).format(_config.format);
        this.pickerObject.$inputBegin.value = val;
      }
      return;
    }
    // 规定天数内
    if (Number.isInteger(Number(_config.between))) {
      const endRangeMoment = endMoment.add(-Number(_config.between), 'day');
      if (endRangeMoment.isAfter(beginMoment)) {
        const val = endRangeMoment.format(_config.format);
        this.pickerObject.$inputBegin.value = val;
      }
    }
  }
}
// 设置日期选择框位置
function setContainerPos(_this) {
  const offset = _this.$target.getBoundingClientRect();
  _this.pickerObject.$container.style.top = `${offset.top + offset.height}px`;
  _this.pickerObject.$container.style.left = `${offset.left}px`;
}
// 修复满足格式但不完全符合的 time 格式修正
function fillTime(_this, $time) {
  const time = $time.value;
  const timeResult = time && time.match(API.timeReg(_this));
  if (!time ||!timeResult) {
    return;
  } else {
    if (timeResult) {
      const format = _this.config.format.split(' ')[1];
      const filledTime = format
      .replace(/HH/, timeResult[1])
      .replace(/mm/, API.fillTime(timeResult[3]))
      .replace(/ss/, API.fillTime(timeResult[5]));
      $time.value = filledTime;
      if (!_this.config.isRange) {
        $time.dispatchEvent(new Event('keyup'));
      }
    }
  }
}
// 修复满足格式但不完全符合的 day 格式修正
function fillDay(_this, $day) {
  const day = $day.value;
  const reg = API.dayReg(_this);
  const dayResult = day.match(reg);
  if (!day ||!dayResult) {
    return;
  } else {
    if (dayResult) {
      // 兼容 201808 变为 2018-00-08 的情况
      dayResult = API.fixedFill(dayResult);
      day = dayResult[1] + _this.splitStr + API.fillTime(dayResult[3]) + _this.splitStr + API.fillTime(dayResult[5]);
      $day.value = day;
      if (!_this.config.isRange) {
        $day.dispatchEvent(new Event('keyup'));
      }
    }
  }
}
// renderSidebar
function rederSidebar(_this) {
  let html = '';
  const options = _this.config.shortcutOptions;
  for (let i = 0; i < options.length; i++) {
    const time = options[i].time || '';
    html += RENDERAPI.sideBarButton(options[i].day, time, options[i].name);
  }
  return RENDERAPI.sideBarTpl(html);
}
// 设置选中值
function setValue(_this, date) {
  _this.$container.querySelectorAll('.c-datepicker-date-table td.current').forEach((td) => td.classList.remove('current'));
  // var date = _moment2.format(_this.config.format);
  const timeArr = date.split(' ');
  _this.$input.value = date;
  _this.$container.querySelector('.c-datePicker__input-day').value = timeArr[0];
  if (timeArr.length > 1) {
    _this.$container.querySelector('.c-datePicker__input-time').value = timeArr[1];
  }
}
// 单个：填充表单时当选的值为空时，自动填充的值
function getMomentWhenEmpty(_this) {
  let _moment, type;
  const momentMin = moment(_this.config.min, _this.config.format);
  const momentMax = moment(_this.config.max, _this.config.format);
  if (_this.config.min && moment().isBefore(momentMin)) {
    _moment = momentMin.format(_this.config.format);
    type = 'min';
  } else if (_this.config.max && moment().isAfter(momentMax)) {
    _moment = momentMax.format(_this.config.format);
    type = 'max';
  } else {
    _moment = moment().format(_this.config.format);
    type = 'active';
  }
  return {
    value: _moment,
    type: type
  };
}
// 定义日期相关的常量对象
const datePickerDates = {
  'zh-CN': {
    days: ["日", "一", "二", "三", "四", "五", "六"],
    months: ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"],
    now: "此刻",
    clear: '清空',
    headerYearLink: '年',
    units: ['年', '月'],
    confirm: '确定',
    cancel: '取消',
    chooseDay: '选择日期',
    chooseTime: '选择时间',
    begin: '开始时间',
    end: '结束时间',
    prevYear: '前一年',
    prevMonth: '上个月',
    nextYear: '后一年',
    nextMonth: '下个月',
    zero: '0 点'
  }
};

// 创建一个自定义指令，类似于 jQuery 的插件方式
const datePickerDirective = {
  mounted(el, binding) {
    // 创建一个新的 DatePicker 实例并传入选项和元素
    new DatePicker(binding.value, el);
  }
};

// 在 Vue 应用中注册自定义指令
const app = createApp({});
app.directive('datePicker', datePickerDirective);

onMounted(() => {
  document.querySelectorAll('.J-datepicker-time').forEach((el) => {
    el.datePicker({
      format: 'HH:mm:ss',
      min: '04:23:11',
    });
  });
  document.querySelectorAll('.J-datepicker-time-range').forEach((el) => {
    el.datePicker({
      format: 'HH:mm:ss',
      isRange: true,
      min: '04:23:11',
      max: '20:59:59',
    });
  });

  const DATAPICKERAPI = {
    activeMonthRange: () => {
      return {
        begin: moment().set({ date: 1, hour: 0, minute: 0, second: 0 }).format('YYYY-MM-DD HH:mm:ss'),
        end: moment().set({ hour: 23, minute: 59, second: 59 }).format('YYYY-MM-DD HH:mm:ss'),
      };
    },
    shortcutMonth: () => {
      const nowDay = moment().get('date');
      const prevMonthFirstDay = moment().subtract(1, 'months').set({ date: 1 });
      const prevMonthDay = moment().diff(prevMonthFirstDay, 'days');
      return {
        now: `-${nowDay},0`,
        prev: `-${prevMonthDay},-${nowDay}`,
      };
    },
  shortcutPrevHours: (hour) => {
  const nowDay = moment().get('date');
  const prevHours = moment().subtract(hour, 'hours');
  const prevDate = prevHours.get('date') - nowDay;
  const nowTime = moment().format('HH:mm:ss');
  const prevTime = prevHours.format('HH:mm:ss');
  return {
    day: `${prevDate},0`,
    time: `${prevTime},${nowTime}`,
    name: `Nearly ${hour} Hours`,
  };
},
    rangeMonthShortcutOption1: () => {
      const result = DATAPICKERAPI.shortcutMonth();
      const resultTime = DATAPICKERAPI.shortcutPrevHours(18);
      return [
        {
          name: 'Yesterday',
          day: '-1,-1',
          time: '00:00:00,23:59:59',
        },
        {
          name: 'This Month',
          day: result.now,
          time: '00:00:00,',
        },
        {
          name: 'Lasy Month',
          day: result.prev,
          time: '00:00:00,23:59:59',
        },
        {
          name: resultTime.name,
          day: resultTime.day,
          time: resultTime.time,
        },
      ];
    },
    rangeShortcutOption1: [
      {
        name: 'Last week',
        day: '-7,0',
      },
      {
        name: 'Last Month',
        day: '-30,0',
      },
      {
        name: 'Last Three Months',
        day: '-90, 0',
      },
    ],
    singleShortcutOptions1: [
      {
        name: 'Today',
        day: '0',
        time: '00:00:00',
      },
      {
        name: 'Yesterday',
        day: '-1',
        time: '00:00:00',
      },
      {
        name: 'One Week Ago',
        day: '-7',
      },
    ],
  };

  document.querySelectorAll('.J-datepicker').forEach((el) => {
    el.datePicker({
      hasShortcut: true,
      min: '2018-01-01 04:00:00',
      max: '2029-10-29 20:59:59',
      shortcutOptions: [
        {
          name: 'Today',
          day: '0',
        },
        {
          name: 'Yesterday',
          day: '-1',
          time: '00:00:00',
        },
        {
          name: 'One Week Ago',
          day: '-7',
        },
      ],
      hide: () => {
        console.info(this);
      },
    });
  });

  document.querySelectorAll('.J-datepicker-day').forEach((el) => {
    el.datePicker({
      hasShortcut: true,
      shortcutOptions: [
        {
          name: 'Today',
          day: '0',
        },
        {
          name: 'Yesterday',
          day: '-1',
        },
        {
          name: 'One week ago',
          day: '-7',
        },
      ],
    });
  });

  document.querySelectorAll('.J-datepicker-range-day').forEach((el) => {
    el.datePicker({
      hasShortcut: true,
      format: 'YYYY-MM-DD',
      isRange: true,
      shortcutOptions: DATAPICKERAPI.rangeShortcutOption1,
    });
  });

  document.querySelectorAll('.J-datepickerTime-single').forEach((el) => {
    el.datePicker({
      format: 'YYYY-MM-DD HH:mm',
    });
  });

  document.querySelectorAll('.J-datepickerTime-range').forEach((el) => {
    el.datePicker({
      format: 'YYYY-MM-DD HH:mm',
      isRange: true,
    });
  });

  document.querySelectorAll('.J-datepicker-range').forEach((el) => {
    el.datePicker({
      hasShortcut: true,
      min: '2018-01-01 06:00:00',
      max: '2029-04-29 20:59:59',
      isRange: true,
      shortcutOptions: [
        {
          name: 'Yesterday',
          day: '-1,-1',
          time: '00:00:00,23:59:59',
        },
        {
          name: 'Last Week',
          day: '-7,0',
          time: '00:00:00,',
        },
        {
          name: 'Last Month',
          day: '-30,0',
          time: '00:00:00,',
        },
        {
          name: 'Last Three Months',
          day: '-90, 0',
          time: '00:00:00,',
        },
      ],
      hide: (type) => {
        console.info(el.querySelectorAll('input')[0].value, el.querySelectorAll('input')[1].value);
        console.info('Type:', type);
      },
    });
  });

  document.querySelectorAll('.J-datepicker-range-betweenMonth').forEach((el) => {
    el.datePicker({
      isRange: true,
      between: 'month',
      hasShortcut: true,
      shortcutOptions: DATAPICKERAPI.rangeMonthShortcutOption1(),
    });
  });

  document.querySelectorAll('.J-datepicker-range-between30').forEach((el) => {
    el.datePicker({
      isRange: true,
      between: 30,
    });
  });

  document.querySelectorAll('.J-yearMonthPicker-single').forEach((el) => {
    el.datePicker({
      format: 'YYYY-MM',
      min: '2018-01',
      max: '2029-04',
      hide: (type) => {
        console.info(el.querySelectorAll('input')[0].value);
      },
    });
  });

  document.querySelectorAll('.J-yearPicker-single').forEach((el) => {
    el.datePicker({
      format: 'YYYY',
      min: '2018',
      max: '2029',
    });
  });
});
</script>

<style>
/* 使用 CSS 模块 */
@font-face {
  font-family: 'kxiconfont';
  src: url('iconfont.eot?t=1534989522363');
  src: url('iconfont.eot?t=1534989522363#iefix') format('embedded-opentype'),
    url('data:application/x-font-woff;charset=utf-8;base64,d09GRgABAAAAAAVIAAsAAAAACDAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAABHU1VCAAABCAAAADMAAABCsP6z7U9TLzIAAAE8AAAARAAAAFY8j0g/Y21hcAAAAYAAAABsAAABuLPUtGhnbHlmAAAB7AAAAUwAAAG8Fx4wJ2hlYWQAAAM4AAAALwAAADYSZ6+haGhlYQAAA2gAAAAcAAAAJAfeA4dobXR4AAADhAAAAA4AAAAYGAAAAGxvY2EAAAOUAAAADgAAAA4BYADIbWF4cAAAA6QAAAAfAAAAIAETAEFuYW1lAAADxAAAAUUAAAJtPlT+fXBvc3QAAAUMAAAAOwAAAE2wPds5eJxjYGRgYOBikGPQYWB0cfMJYeBgYGGAAJAMY05meiJQDMoDyrGAaQ4gZoOIAgCKIwNPAHicY2BkYWCcwMDKwMHUyXSGgYGhH0IzvmYwYuRgYGBiYGVmwAoC0lxTGByeST7zZW7438AQw9zA0AAUZgTJAQDk6QxJeJztkdENgCAMRA8BYwwL+IMM4ATOIGv45dxdA9urY3jkNddL048CIAOIyqEkIDwIMN2aBuYRK/OEU/tF3wTIJk36GHTVHRU4Uegm3R65dcavwnp9XbbLOXZnqQ5vuzucaY79kXQH6QUQEheSeJxdjj9Lw1AUxd9JmsSQVKtNG4ekSrXp1kATG0HtEKij+AWEIhgUXJxcHdx0dqj45yOog6OFqoOD4OLQb+BQN3Hz6nuW0uobHvf33jnnHgbGj9STnpnCmI5AxxwsNPfoDDv72EWTLulCSIRO3pZ6bIxNCqWmw0YtLA0GaZpufCzRvY91E4t0PYRzrPn0QB0O6dn0KAxz33nulMiNdJShWspgsPEmHI9Y8enKpKcRMLApdi6jzokn/4F+5+8PqSubTGUaYzzut6vU9ajlIRHXMQ48bJWoRa0SjkQV7rmTO3LMDDbPQu4ah1aAXUdUQVbV1GK5grIXRvyhVrX5T97SVB8LoVdUXU5BNap5EmtTKkXtNikKrQZx46QRJ67juMlwbN7mZ/KvhpE1dUWOFSHuW77qgeO6TvLftXH4ksnlMp+n2kS6wKv+ANtFbjx4nGNgZGBgAOJlq92M4/ltvjJwszCAwPUlppcQ9P99LAzMTkAuBwMTSBQALeQKjwB4nGNgZGBgbvjfwBDDwgACQJKRARWwAQBHDAJveJxjYWBgYMGCAQFoABkAAAAAAAAAFABAAG4AiADeAAB4nGNgZGBgYGMwZWBmAAEmIOYCQgaG/2A+AwAOXgFVAHicZY9NTsMwEIVf+gekEqqoYIfkBWIBKP0Rq25YVGr3XXTfpk6bKokjx63UA3AejsAJOALcgDvwSCebNpbH37x5Y08A3OAHHo7fLfeRPVwyO3INF7gXrlN/EG6QX4SbaONVuEX9TdjHM6bCbXRheYPXuGL2hHdhDx18CNdwjU/hOvUv4Qb5W7iJO/wKt9Dx6sI+5l5XuI1HL/bHVi+cXqnlQcWhySKTOb+CmV7vkoWt0uqca1vEJlODoF9JU51pW91T7NdD5yIVWZOqCas6SYzKrdnq0AUb5/JRrxeJHoQm5Vhj/rbGAo5xBYUlDowxQhhkiMro6DtVZvSvsUPCXntWPc3ndFsU1P9zhQEC9M9cU7qy0nk6T4E9XxtSdXQrbsuelDSRXs1JErJCXta2VELqATZlV44RelzRiT8oZ0j/AAlabsgAAAB4nGNgYoAALgbsgI2RiZGZkYWRlZGNkZ2BJSc1rYQlJ7G4hDUtswhIFmWmZ5SwJufkJ2czMAAApPgJ/AA=')
      format('woff'),
    url('iconfont.ttf?t=1534989522363') format('truetype'),
    url('iconfont.svg?t=1534989522363#iconfont') format('svg');
}

.kxiconfont {
  font-family: 'kxiconfont'!important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.icon-left:before {
  content: '\e619';
}

.icon-last:before {
  content: '\e61e';
}

.icon-first:before {
  content: '\e620';
}

.icon-right:before {
  content: '\e64d';
}

.icon-clock:before {
  content: '\e61f';
}

a {
  color: #20a0ff;
}

*:before,
*:after {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

.c-datepicker-picker {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  width: 296px;
  position: absolute;
  z-index: 2001;
}

.c-datepicker-picker *:before,
.c-datepicker-picker *:after {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}

.c-datepicker-picker {
  color: #333;
  border: 1px solid #e4e7ed;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  background: #fff;
  border-radius: 4px;
  line-height: 30px;
  margin: 5px 0;
  display: none;
}

.c-datepicker-picker a {
  color: #333;
}

.c-datepicker-picker.only-time {
  width: 150px;
}

.c-datepicker-picker.only-time.c-datepicker-time-panel {
  position: relative;
  box-shadow: none;
  margin: 0;
}

.c-datepicker-picker.only-time.c-datepicker-date-picker__time-header,
.c-datepicker-picker.only-time.c-datepicker-date-picker__editor-wrap {
  padding: 0;
}

.c-datepicker-date-picker.has-time.c-datepicker-picker__body-wrapper {
  position: relative;
}

.c-datepicker-date-picker__time-header {
  position: relative;
  border-bottom: 1px solid #e4e4e4;
  font-size: 12px;
  padding: 8px 5px 5px;
  display: table;
  width: 100%;
  box-sizing: border-box;
}

.c-datepicker-date-picker__editor-wrap {
  position: relative;
  display: table-cell;
  padding: 0 5px;
}

.c-datepicker-input {
  position: relative;
  font-size: 14px;
  display: inline-block;
  width: 100%;
}

.c-datepicker-input--small {
  font-size: 13px;
}

.c-datepicker-input__inner {
  -webkit-appearance: none;
  background-color: #fff;
  background-image: none;
  border-radius: 4px;
  border: 1px solid #dcdfe6;
  box-sizing: border-box;
  color: #333;
  display: inline-block;
  font-size: inherit;
  height: 40px;
  line-height: 40px;
  outline: none;
  width: 100%;
  padding: 0 10px;
}

.c-datepicker-input--small.c-datepicker-input__inner {
  height: 32px;
  line-height: 32px;
}

.c-datepicker-time-panel {
  margin: 5px 0;
  border: 1px solid #e4e7ed;
  background-color: #fff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  border-radius: 2px;
  position: absolute;
  width: 180px;
  left: 0;
  z-index: 1000;
  user-select: none;
}

.c-datepicker-time-panel__content {
  font-size: 0;
  position: relative;
  overflow: hidden;
}

.c-datepicker-time-panel__content:before,
.c-datepicker-time-panel__content:after {
  content: '';
  top: 50%;
  position: absolute;
  margin-top: -15px;
  height: 32px;
  z-index: -1;
  left: 0;
  right: 0;
  box-sizing: border-box;
  padding-top: 6px;
  text-align: left;
  border-top: 1px solid #e4e7ed;
  border-bottom: 1px solid #e4e7ed;
}

.c-datepicker-time-panel__content:after {
  left: 50%;
  margin-left: 12%;
  margin-right: 12%;
}

.c-datepicker-time-panel__content.has-seconds:after {
  left: 66.66667%;
}

.c-datepicker-time-spinner {
  width: 100%;
  white-space: nowrap;
}

.c-datepicker-time-spinner.has-seconds.c-datepicker-time-spinner__wrapper {
  width: 33.3%;
}

.c-datepicker-time-spinner__wrapper {
  height: 190px;
  overflow: auto;
  display: inline-block;
  width: 50%;
  vertical-align: top;
  position: relative;
}

.c-datepicker-scrollbar {
  overflow: hidden;
  position: relative;
}

.c-datepicker-scrollbar::-webkit-scrollbar {
  display: none;
}

.c-datepicker-scrollbar__wrap {
  overflow: scroll;
  height: 100%;
}

.c-datepicker-time-spinner__wrapper.c-datepicker-scrollbar__wrap:not(.c-datepicker-scrollbar__wrap--hidden-default) {
  padding-bottom: 15px;
}

.c-datepicker-time-spinner__list {
  margin: 0;
  list-style: none;
}

.c-datepicker-time-spinner__input.c-datepicker-input.c-datepicker-input__inner,
.c-datepicker-time-spinner__list {
  padding: 0;
  text-align: center;
}

.c-datepicker-time-spinner__list:after,
.c-datepicker-time-spinner__list:before {
  content: '';
  display: block;
  width: 100%;
  height: 80px;
}

.c-datepicker-time-spinner__item {
  height: 32px;
  line-height: 32px;
  font-size: 12px;
  color: #333;
}

.c-datepicker-time-spinner__item.disabled {
  color: #999;
}

.c-datepicker-time-spinner__item.disabled:hover {
  cursor: no-drop;
}

.c-datepicker-scrollbar__bar.is-horizontal {
  height: 6px;
  left: 2px;
}

.c-datepicker-scrollbar__bar.is-horizontal>div {
  height: 100%;
}

.c-datepicker-scrollbar__thumb {
  position: relative;
  display: block;
  width: 0;
  height: 0;
  cursor: pointer;
  border-radius: inherit;
  background-color: rgba(144, 146, 152, 0.3);
  transition: background-color 0.3s;
}

.c-datepicker-time-panel__footer {
  border-top: 1px solid #e4e4e4;
  padding: 4px 0;
  height: 36px;
  line-height: 25px;
  text-align: right;
  box-sizing: border-box;
}

.c-datepicker-time-panel__btn {
  border: none;
  line-height: 28px;
  padding: 0 5px;
  margin: 0 5px;
  cursor: pointer;
  background-color: transparent;
  outline: none;
  font-size: 12px;
  color: #303133;
}

.c-datepicker-time-panel__btn.confirm {
  font-weight: 800;
  color: #409eff;
}

.c-datepicker-time-panel__btn.cancel {
  font-weight: 800;
  color: #999;
}

.c-datepicker-time-panel__btn.min,
.c-datepicker-time-panel__btn.max {
  float: left;
  padding-right: 0;
}

.c-datepicker-date-picker__header {
  margin: 10px 12px;
  text-align: center;
}

.c-datepicker-date-picker__header-label {
  font-size: 16px;
  font-weight: 500;
  padding: 0 5px;
  line-height: 22px;
  text-align: center;
  cursor: pointer;
  color: #333;
}

.c-datepicker-picker__content {
  position: relative;
  margin: 10px 15px;
}

.c-datepicker-date-picker table {
  table-layout: fixed;
  width: 100%;
}

.c-datepicker-date-table {
  font-size: 12px;
  user-select: none;
  border-spacing: 0;
  width: 100%;
}

.c-datepicker-date-table td.next-month.cell,
.c-datepicker-date-table td.prev-month.cell {
  color: #c0c4cc;
}

.c-datepicker-date-table td.disabled.cell,
.c-datepicker-month-table td.disabled.cell,
.c-datepicker-year-table td.disabled.cell {
  text-decoration: line-through;
  color: #999;
}

.c-datepicker-year-table td.current.cell {
  color: #409eff;
}

.c-datepicker-date-table td {
  width: 34px;
  height: 30px;
  padding: 4px 0;
  box-sizing: border-box;
  text-align: center;
  cursor: pointer;
  position: relative;
}

.c-datepicker-date-table td div {
  height: 30px;
  padding: 3px 0;
  box-sizing: border-box;
}

.c-datepicker-date-table td.cell {
  width: 24px;
  height: 24px;
  display: block;
  margin: 0 auto;
  line-height: 24px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  border-radius: 50%;
}

.c-datepicker-date-table td.in-range div,
.c-datepicker-date-table td.in-range div:hover {
  background-color: #f2f6fc;
}

.c-datepicker-date-table td.end-date div {
    margin-right: 1px;
    border-top-right-radius: 15px;
    border-bottom-right-radius: 15px;
}

.c-datepicker-date-table th {
    padding: 5px;
    color: #333;
    font-weight: 400;
    border-bottom: 1px solid #ebeef5;
}

.c-datepicker-year-table,
.c-datepicker-month-table {
    font-size: 14px;
    margin: -1px;
    border-collapse: collapse;
    margin-bottom: 5px;
}

.c-datepicker-month-table td.cell {
    width: 48px;
    height: 32px;
    display: block;
    line-height: 32px;
    color: #333;
    margin: 0 auto;
}

.c-datepicker-button {
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    background: #fff;
    border: 1px solid #dcdfe6;
    border-color: #dcdfe6;
    color: #333;
    -webkit-appearance: none;
    text-align: center;
    box-sizing: border-box;
    outline: none;
    margin: 0;
    transition:.1s;
    font-weight: 500;
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    padding: 12px 20px;
    font-size: 14px;
    border-radius: 4px;
}

.c-datepicker-button--mini {
    padding: 7px 15px;
    font-size: 12px;
    border-radius: 3px;
}

.c-datepicker-button--text {
    border-color: transparent;
    color: #409eff;
    background: transparent;
    padding-left: 0;
    padding-right: 0;
}

.c-datepicker-picker__btn-cancel {
    color: #999;
}

.popper__arrow {
    top: -6px;
    left: 50%;
    margin-right: 3px;
    border-top-width: 0;
    border-bottom-color: #ebeef5;
    position: absolute;
    display: block;
    width: 0;
    height: 0;
    border-color: transparent;
    border-style: solid;
}

.popper__arrow {
    border-width: 6px;
    filter: drop-shadow(0 2px 12px rgba(0, 0, 0, 0.03));
}

.c-datepicker-date-table td.available:hover {
    color: #409eff;
}

.c-datepicker-picker__footer {
    border-top: 1px solid #e4e4e4;
    padding: 4px;
    text-align: right;
    background-color: #fff;
    position: relative;
    font-size: 0;
}

.c-datepicker-button+.c-datepicker-button {
    margin-left: 10px;
}

.c-datepicker-picker__icon-btn {
    cursor: pointer;
    padding: 2px 7px;
}

.c-datepicker-picker__icon-btn:hover {
    color: #409eff;
}

.c-datepicker-date-table td.in-range div,
.c-datepicker-date-table td.in-range div:hover {
    background-color: #f2f6fc;
}

.c-datepicker-scrollbar__thumb {
    position: relative;
    display: block;
    width: 0;
    height: 0;
    cursor: pointer;
    border-radius: inherit;
    background-color: rgba(144, 146, 152, 0.3);
    transition: background-color.3s;
}

.c-datepicker-scrollbar__bar.is-vertical>div {
    width: 100%;
}

.c-datepicker-scrollbar__bar {
    position: absolute;
    right: 2px;
    bottom: 2px;
    z-index: 1;
    border-radius: 4px;
    opacity: 0;
    transition: opacity.12s ease-out;
}

.c-datepicker-date-table td.today.cell {
    color: #409eff;
    font-weight: 700;
}

.c-datepicker-date-table td.cell {
    width: 24px;
    height: 24px;
    display: block;
    margin: 0 auto;
    line-height: 24px;
    border-radius: 50%;
}

.c-datepicker-date-table td.current:not(.disabled).cell {
    color: #fff;
    background-color: #409eff;
}

.c-datepicker-month-table td.cell:hover,
.c-datepicker-month-table td.current:not(.disabled).cell {
    color: #409eff;
}

.c-datepicker-scrollbar__bar.is-vertical {
    width: 6px;
    top: 2px;
}

.c-datepicker-scrollbar__bar.is-horizontal {
    height: 6px;
    left: 2px;
}

.c-datepicker-scrollbar__bar.is-horizontal>div {
    height: 100%;
}

.c-datepicker-month-table td,
.c-datepicker-year-table td {
    text-align: center;
    padding: 16px 3px;
    cursor: pointer;
}

.c-datepicker-picker [slot=sidebar]+.c-datepicker-picker__body,
.c-datepicker-picker__sidebar+.c-datepicker-picker__body {
    margin-left: 102px;
}

.c-datepicker-picker [slot=sidebar],
.c-datepicker-picker__sidebar {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 102px;
    border-right: 1px solid #e4e4e4;
    box-sizing: border-box;
    padding-top: 6px;
    background-color: #fff;
    overflow: auto;
}

.c-datepicker-picker__shortcut {
    display: block;
    width: 100%;
    border: 0;
    background-color: transparent;
    line-height: 28px;
    font-size: 14px;
    color: #333;
    padding-left: 8px;
    text-align: left;
    outline: none;
    cursor: pointer;
}

.c-datepicker-picker__shortcut:hover {
    color: #409eff;
}

.c-datepicker-date-range-picker.has-sidebar {
    width: 636px;
}

.c-datepicker-date-picker.has-sidebar {
    width: 390px;
}

.c-datepicker-picker__body-wrapper:after,
.c-datepicker-picker__body:after,
.c-datepicker-picker__body-content:after {
    content: "";
    display: table;
    clear: both;
}

.c-datepicker-date-range-picker {
    width: 532px;
}

.c-datepicker-only-time-title {
    text-align: center;
}

.c-datepicker-date-range-picker.only-time {
    width: 400px;
}

.c-datepicker-date-range-picker.only-time.c-datepicker-time-panel {
    position: relative;
}

.c-datepicker-date-range-picker.only-time.c-datepicker-time-panel {
    box-shadow: none;
}

.c-datepicker-date-range-picker-panel__wrap {
    float: left;
    width: 50%;
    box-sizing: border-box;
    margin: 0;
    padding: 10px;
}

.c-datepicker-date-range-picker-panel__wrap.is-left {
    border-right: 1px solid #e4e4e4;
}

.c-datepicker-picker__content {
    margin: 0;
    width: 100%;
    margin: 0 auto;
}

.c-datepicker-date-range-picker__time-header {
    position: relative;
    border-bottom: 1px solid #e4e4e4;
    font-size: 12px;
    padding: 8px 5px 5px;
    display: table;
    width: 100%;
    box-sizing: border-box;
}

.c-datepicker-date-range-picker__time-content {
    box-sizing: border-box;
    display: table-cell;
}

.c-datepicker-date-range-picker__editor-wrap {
    position: relative;
    display: table-cell;
    padding: 0 5px;
}

.is-right.c-datepicker-date-range-picker__header-year {
    margin-left: 66px;
    margin-right: 10px;
}

.is-right.c-datepicker-date-range-picker__header-month {
    margin-right: 10px;
}

.is-left.c-datepicker-date-range-picker__header-year {
    margin-left: 12px;
}

.is-left.c-datepicker-date-range-picker__header-month {
    margin-left: 10px;
}

.c-datepicker-date-range-picker__header {
    user-select: none;
    font-size: 16px;
}

.c-datepicker-data-input {
    border: none;
    outline: none;
    display: inline-block;
    height: 100%;
    margin: 0;
    padding: 0;
    text-align: center;
    font-size: 14px;
    color: #333;
    line-height: 1;
    vertical-align: top;
    background-color: #fff;
}

div.c-datepicker-data-input {
    line-height: 34px;
}

.c-datepicker-data-input.only-date {
    width: 86px;
}

.c-datepicker-data-input.only-time {
    width: 60px;
}

.c-datepicker-single-editor.c-datepicker-data-input {
    text-align: left;
}

.c-datepicker-range-separator {
    padding: 0 3px;
    display: inline-block;
    text-align: center;
}

.c-datepicker-date-editor {
    -webkit-appearance: none;
    background-color: #fff;
    background-image: none;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    box-sizing: border-box;
    color: #333;
    display: inline-block;
    font-size: inherit;
    height: 34px;
    line-height: 34px;
    outline: none;
    padding: 0 15px;
    white-space: nowrap;
}

.c-datepicker-range__icon {
    display: inline-block;
    text-align: center;
}

.is-en.c-datepicker-date-range-picker.is-right.c-datepicker-date-range-picker__header-year {
    margin-left: 93px;
}

.is-en.c-datepicker-date-range-picker.is-right.c-datepicker-date-range-picker__header-month {
    margin-right: 22px;
}

.is-en.c-datepicker-date-range-picker.is-left.c-datepicker-date-range-picker__header-year {
    margin-left: 35px;
}

.is-en.c-datepicker-time-panel__btn {
    margin: 0;
}

</style>
