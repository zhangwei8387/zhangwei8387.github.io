---
title: 日历
date: 2025-12-03 15:54:00
layout: page
---

<div id="calendar-container">
  <style>
    #calendar-container {
      max-width: 900px;
      margin: 20px auto;
      padding: 20px;
    }
    
    .calendar {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      padding: 20px;
    }
    
    .calendar-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 2px solid #2f4154;
    }
    
    .calendar-header h2 {
      margin: 0;
      color: #2f4154;
      font-size: 24px;
    }
    
    .calendar-nav {
      display: flex;
      gap: 10px;
    }
    
    .calendar-nav button {
      background: #2f4154;
      color: white;
      border: none;
      padding: 8px 16px;
      border-radius: 4px;
      cursor: pointer;
      font-size: 14px;
      transition: background 0.3s;
    }
    
    .calendar-nav button:hover {
      background: #1f3144;
    }
    
    .calendar-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 10px;
      margin-top: 20px;
    }
    
    .calendar-day-header {
      text-align: center;
      font-weight: bold;
      padding: 10px;
      color: #2f4154;
      background: #f5f5f5;
      border-radius: 4px;
    }
    
    .calendar-day {
      aspect-ratio: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 10px;
      border: 1px solid #e0e0e0;
      border-radius: 4px;
      cursor: pointer;
      transition: all 0.3s;
      position: relative;
      min-height: 80px;
    }
    
    .calendar-day:hover {
      background: #f0f4f8;
      transform: translateY(-2px);
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    
    .calendar-day.other-month {
      opacity: 0.3;
    }
    
    .calendar-day.today {
      background: #2f4154;
      color: white;
      font-weight: bold;
    }
    
    .calendar-day.has-posts {
      background: #e3f2fd;
      border-color: #2196f3;
    }
    
    .calendar-day.has-posts::after {
      content: '';
      position: absolute;
      bottom: 5px;
      width: 6px;
      height: 6px;
      background: #2196f3;
      border-radius: 50%;
    }
    
    .calendar-day.today.has-posts::after {
      background: white;
    }
    
    .day-number {
      font-size: 18px;
    }
    
    .post-count {
      font-size: 11px;
      margin-top: 4px;
      color: #666;
    }
    
    .calendar-day.today .post-count {
      color: white;
    }
    
    .posts-list {
      margin-top: 30px;
      padding-top: 20px;
      border-top: 2px solid #e0e0e0;
    }
    
    .posts-list h3 {
      color: #2f4154;
      margin-bottom: 15px;
    }
    
    .post-item {
      padding: 15px;
      margin-bottom: 10px;
      background: #f9f9f9;
      border-left: 4px solid #2f4154;
      border-radius: 4px;
      transition: all 0.3s;
    }
    
    .post-item:hover {
      background: #f0f4f8;
      transform: translateX(5px);
    }
    
    .post-item a {
      color: #2f4154;
      text-decoration: none;
      font-size: 16px;
      font-weight: 500;
    }
    
    .post-item a:hover {
      color: #0366d6;
    }
    
    .post-date {
      color: #666;
      font-size: 14px;
      margin-top: 5px;
    }
    
    @media (max-width: 768px) {
      .calendar-grid {
        gap: 5px;
      }
      
      .calendar-day {
        min-height: 60px;
        padding: 5px;
      }
      
      .day-number {
        font-size: 14px;
      }
      
      .post-count {
        font-size: 9px;
      }
    }
  </style>
  
  <div class="calendar">
    <div class="calendar-header">
      <h2 id="current-month-year"></h2>
      <div class="calendar-nav">
        <button id="prev-month">上个月</button>
        <button id="today-btn">今天</button>
        <button id="next-month">下个月</button>
      </div>
    </div>
    
    <div class="calendar-grid" id="calendar-days-header">
      <div class="calendar-day-header">日</div>
      <div class="calendar-day-header">一</div>
      <div class="calendar-day-header">二</div>
      <div class="calendar-day-header">三</div>
      <div class="calendar-day-header">四</div>
      <div class="calendar-day-header">五</div>
      <div class="calendar-day-header">六</div>
    </div>
    
    <div class="calendar-grid" id="calendar-days"></div>
    
    <div class="posts-list" id="posts-list" style="display: none;">
      <h3 id="selected-date"></h3>
      <div id="posts-content"></div>
    </div>
  </div>
</div>

<script>
(function() {
  let currentDate = new Date();
  let selectedDate = null;
  
  // 文章数据数组
  // 数据格式示例: 
  // [
  //   { date: '2025-12-03', title: '文章标题', url: '/2025/12/03/article-name/' },
  //   { date: '2025-12-01', title: '另一篇文章', url: '/2025/12/01/another-article/' }
  // ]
  // 可以通过 Hexo 的 EJS/Nunjucks 模板引擎注入数据，例如：
  // <%- JSON.stringify(site.posts.sort('date', -1).map(post => ({
  //   date: date(post.date, 'YYYY-MM-DD'),
  //   title: post.title,
  //   url: url_for(post.path)
  // }))) %>
  const posts = [];
  
  function formatDate(date) {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}-${month}-${day}`;
  }
  
  function getMonthData(year, month) {
    const firstDay = new Date(year, month, 1);
    const lastDay = new Date(year, month + 1, 0);
    const firstDayOfWeek = firstDay.getDay();
    const daysInMonth = lastDay.getDate();
    
    return {
      firstDay,
      lastDay,
      firstDayOfWeek,
      daysInMonth
    };
  }
  
  function getPostsForDate(date) {
    const dateStr = formatDate(date);
    return posts.filter(post => post.date === dateStr);
  }
  
  function renderCalendar() {
    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();
    const monthNames = ['一月', '二月', '三月', '四月', '五月', '六月', 
                        '七月', '八月', '九月', '十月', '十一月', '十二月'];
    
    document.getElementById('current-month-year').textContent = 
      `${year}年 ${monthNames[month]}`;
    
    const { firstDayOfWeek, daysInMonth } = getMonthData(year, month);
    const calendarDays = document.getElementById('calendar-days');
    calendarDays.innerHTML = '';
    
    // Previous month days
    const prevMonthData = getMonthData(year, month - 1);
    for (let i = firstDayOfWeek - 1; i >= 0; i--) {
      const day = prevMonthData.daysInMonth - i;
      const dayDiv = createDayElement(year, month - 1, day, true);
      calendarDays.appendChild(dayDiv);
    }
    
    // Current month days
    const today = new Date();
    for (let day = 1; day <= daysInMonth; day++) {
      const date = new Date(year, month, day);
      const isToday = date.toDateString() === today.toDateString();
      const dayDiv = createDayElement(year, month, day, false, isToday);
      calendarDays.appendChild(dayDiv);
    }
    
    // Next month days
    const remainingDays = 42 - (firstDayOfWeek + daysInMonth);
    for (let day = 1; day <= remainingDays; day++) {
      const dayDiv = createDayElement(year, month + 1, day, true);
      calendarDays.appendChild(dayDiv);
    }
  }
  
  function createDayElement(year, month, day, isOtherMonth, isToday = false) {
    const dayDiv = document.createElement('div');
    dayDiv.className = 'calendar-day';
    
    if (isOtherMonth) {
      dayDiv.classList.add('other-month');
    }
    
    if (isToday) {
      dayDiv.classList.add('today');
    }
    
    const date = new Date(year, month, day);
    const postsForDay = getPostsForDate(date);
    
    if (postsForDay.length > 0) {
      dayDiv.classList.add('has-posts');
    }
    
    const dayNumber = document.createElement('div');
    dayNumber.className = 'day-number';
    dayNumber.textContent = day;
    dayDiv.appendChild(dayNumber);
    
    if (postsForDay.length > 0) {
      const postCount = document.createElement('div');
      postCount.className = 'post-count';
      postCount.textContent = `${postsForDay.length}篇`;
      dayDiv.appendChild(postCount);
    }
    
    dayDiv.addEventListener('click', () => {
      selectedDate = date;
      showPostsForDate(date, postsForDay);
    });
    
    return dayDiv;
  }
  
  function showPostsForDate(date, postsForDay) {
    const postsList = document.getElementById('posts-list');
    const selectedDateEl = document.getElementById('selected-date');
    const postsContent = document.getElementById('posts-content');
    
    selectedDateEl.textContent = `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日的文章`;
    
    if (postsForDay.length === 0) {
      postsContent.innerHTML = '<p style="color: #666;">这一天没有发表文章</p>';
    } else {
      postsContent.innerHTML = postsForDay.map(post => `
        <div class="post-item">
          <a href="${post.url}">${post.title}</a>
          <div class="post-date">${post.date}</div>
        </div>
      `).join('');
    }
    
    postsList.style.display = 'block';
    postsList.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }
  
  function prevMonth() {
    currentDate.setMonth(currentDate.getMonth() - 1);
    renderCalendar();
  }
  
  function nextMonth() {
    currentDate.setMonth(currentDate.getMonth() + 1);
    renderCalendar();
  }
  
  function goToToday() {
    currentDate = new Date();
    renderCalendar();
  }
  
  // Event listeners
  document.getElementById('prev-month').addEventListener('click', prevMonth);
  document.getElementById('next-month').addEventListener('click', nextMonth);
  document.getElementById('today-btn').addEventListener('click', goToToday);
  
  // Initial render
  renderCalendar();
  
  // 如果页面通过全局变量传入 Hexo 的文章数据，可以在这里初始化
  // 数据格式: window.hexoPostsData = [{ date: 'YYYY-MM-DD', title: '标题', url: '/path/' }, ...]
  if (typeof window.hexoPostsData !== 'undefined' && Array.isArray(window.hexoPostsData)) {
    posts.push(...window.hexoPostsData);
    renderCalendar();
  }
})();
</script>

## 关于日历

这个日历展示了博客文章的发布时间。您可以：

- 点击日期查看当天发布的文章
- 使用导航按钮浏览不同的月份
- 有蓝色圆点标记的日期表示该天有文章发布

> 注意：当前日历是一个基础实现。如果需要显示实际的博客文章数据，需要配置 Hexo 主题以传递文章数据到此页面。
