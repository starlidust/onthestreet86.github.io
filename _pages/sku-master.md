---
title: "SKU 마스터"
permalink: /sku-master/
layout: single
author_profile: false
---

<div id="sku-master">

  <div class="sku-toolbar">
    <button id="btn-view-all" class="btn btn--primary btn--small active">전체 보기</button>
    <button id="btn-view-grouped" class="btn btn--primary btn--small">그룹별 보기</button>
    <button id="btn-view-ungrouped" class="btn btn--primary btn--small">미배정 SKU</button>
    <span class="sku-toolbar__divider">|</span>
    <button id="btn-add-group" class="btn btn--success btn--small">+ 새 그룹 추가</button>
  </div>

  <!-- 그룹 편집 모달 -->
  <div id="group-modal" class="sku-modal" style="display:none;">
    <div class="sku-modal__content">
      <h3 id="modal-title">새 그룹 추가</h3>
      <div class="sku-form-row">
        <label for="modal-group-name">그룹명</label>
        <input type="text" id="modal-group-name" placeholder="그룹명을 입력하세요" />
      </div>
      <div class="sku-form-row">
        <label for="modal-group-desc">설명</label>
        <input type="text" id="modal-group-desc" placeholder="그룹 설명을 입력하세요" />
      </div>
      <div class="sku-modal__actions">
        <button id="modal-save" class="btn btn--primary btn--small">저장</button>
        <button id="modal-cancel" class="btn btn--small">취소</button>
      </div>
    </div>
  </div>

  <!-- 전체 / 그룹별 보기 -->
  <div id="view-all" class="sku-view">
    <h2>전체 SKU 목록</h2>
    <table class="sku-table">
      <thead>
        <tr>
          <th>SKU</th>
          <th>상품명</th>
          <th>카테고리</th>
          <th>그룹</th>
          <th>그룹 변경</th>
        </tr>
      </thead>
      <tbody id="all-sku-tbody">
      </tbody>
    </table>
  </div>

  <div id="view-grouped" class="sku-view" style="display:none;">
    <h2>그룹별 SKU 목록</h2>
    <div id="group-list"></div>
  </div>

  <div id="view-ungrouped" class="sku-view" style="display:none;">
    <h2>미배정 SKU</h2>
    <table class="sku-table">
      <thead>
        <tr>
          <th>SKU</th>
          <th>상품명</th>
          <th>카테고리</th>
          <th>그룹 배정</th>
        </tr>
      </thead>
      <tbody id="ungrouped-sku-tbody">
      </tbody>
    </table>
  </div>

</div>

<style>
#sku-master {
  font-family: inherit;
}
.sku-toolbar {
  margin-bottom: 1.5em;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5em;
  align-items: center;
}
.sku-toolbar__divider {
  color: #ccc;
  font-size: 1.2em;
  margin: 0 0.25em;
}
.sku-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1.5em;
}
.sku-table th,
.sku-table td {
  padding: 0.6em 0.8em;
  border: 1px solid #e0e0e0;
  text-align: left;
}
.sku-table th {
  background: #f5f5f5;
  font-weight: bold;
}
.sku-table tr:hover td {
  background: #fafafa;
}
.sku-group-card {
  border: 1px solid #ddd;
  border-radius: 6px;
  margin-bottom: 1.5em;
  overflow: hidden;
}
.sku-group-card__header {
  background: #f0f4f8;
  padding: 0.75em 1em;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-weight: bold;
  font-size: 1em;
}
.sku-group-card__header .group-actions {
  display: flex;
  gap: 0.4em;
}
.sku-group-card__body {
  padding: 0;
}
.sku-badge {
  display: inline-block;
  padding: 0.15em 0.6em;
  border-radius: 3px;
  font-size: 0.78em;
  font-weight: bold;
  background: #e8f0fe;
  color: #1a73e8;
  margin-right: 0.3em;
}
.sku-badge--ungrouped {
  background: #fce8e6;
  color: #d93025;
}
.sku-modal {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.4);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
}
.sku-modal__content {
  background: #fff;
  border-radius: 8px;
  padding: 2em;
  min-width: 320px;
  max-width: 480px;
  width: 100%;
  box-shadow: 0 4px 24px rgba(0,0,0,0.15);
}
.sku-modal__content h3 {
  margin-top: 0;
}
.sku-form-row {
  margin-bottom: 1em;
  display: flex;
  flex-direction: column;
  gap: 0.3em;
}
.sku-form-row label {
  font-weight: bold;
  font-size: 0.9em;
}
.sku-form-row input, .sku-form-row select {
  padding: 0.4em 0.6em;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1em;
}
.sku-modal__actions {
  display: flex;
  gap: 0.5em;
  justify-content: flex-end;
  margin-top: 1.5em;
}
select.sku-group-select {
  padding: 0.25em 0.4em;
  font-size: 0.9em;
}
</style>

<script>
(function() {
  // SKU 데이터 (Jekyll 데이터에서 로드)
  var skuData = {
    groups: [
      {% for group in site.data.skus.groups %}
      { id: "{{ group.id }}", name: "{{ group.name }}", description: "{{ group.description }}" }{% unless forloop.last %},{% endunless %}
      {% endfor %}
    ],
    items: [
      {% for item in site.data.skus.items %}
      { sku: "{{ item.sku }}", name: "{{ item.name }}", category: "{{ item.category }}", group: "{{ item.group }}" }{% unless forloop.last %},{% endunless %}
      {% endfor %}
    ]
  };

  var groups = skuData.groups.slice();
  var items = skuData.items.map(function(i) { return Object.assign({}, i); });

  function getGroupName(groupId) {
    if (!groupId) return '';
    var g = groups.find(function(g) { return g.id === groupId; });
    return g ? g.name : groupId;
  }

  function buildGroupSelect(selectedGroupId, includeEmpty) {
    var html = '<select class="sku-group-select">';
    if (includeEmpty) html += '<option value="">-- 미배정 --</option>';
    groups.forEach(function(g) {
      var sel = g.id === selectedGroupId ? ' selected' : '';
      html += '<option value="' + g.id + '"' + sel + '>' + g.name + '</option>';
    });
    html += '</select>';
    return html;
  }

  function renderAllView() {
    var tbody = document.getElementById('all-sku-tbody');
    var rows = '';
    items.forEach(function(item, idx) {
      var groupName = getGroupName(item.group);
      var badge = groupName
        ? '<span class="sku-badge">' + groupName + '</span>'
        : '<span class="sku-badge sku-badge--ungrouped">미배정</span>';
      rows += '<tr>' +
        '<td>' + item.sku + '</td>' +
        '<td>' + item.name + '</td>' +
        '<td>' + item.category + '</td>' +
        '<td>' + badge + '</td>' +
        '<td>' + buildGroupSelect(item.group, true) + ' <button class="btn btn--primary btn--small" data-idx="' + idx + '" data-action="assign">배정</button></td>' +
        '</tr>';
    });
    tbody.innerHTML = rows;

    tbody.querySelectorAll('[data-action="assign"]').forEach(function(btn) {
      btn.addEventListener('click', function() {
        var idx = parseInt(btn.getAttribute('data-idx'));
        var sel = btn.parentNode.querySelector('.sku-group-select');
        items[idx].group = sel.value;
        renderAllView();
        if (document.getElementById('view-grouped').style.display !== 'none') renderGroupedView();
        if (document.getElementById('view-ungrouped').style.display !== 'none') renderUngroupedView();
      });
    });
  }

  function renderGroupedView() {
    var container = document.getElementById('group-list');
    var html = '';
    groups.forEach(function(g) {
      var groupItems = items.filter(function(i) { return i.group === g.id; });
      html += '<div class="sku-group-card">' +
        '<div class="sku-group-card__header">' +
        '<span>' + g.name + ' <small style="font-weight:normal;color:#555;">(' + (g.description || '') + ')</small></span>' +
        '<span class="group-actions">' +
        '<button class="btn btn--small" data-group-id="' + g.id + '" data-action="edit-group">편집</button>' +
        '<button class="btn btn--danger btn--small" data-group-id="' + g.id + '" data-action="delete-group">삭제</button>' +
        '</span>' +
        '</div>' +
        '<div class="sku-group-card__body">';

      if (groupItems.length === 0) {
        html += '<p style="padding:0.8em 1em;color:#888;margin:0;">이 그룹에 배정된 SKU가 없습니다.</p>';
      } else {
        html += '<table class="sku-table"><thead><tr><th>SKU</th><th>상품명</th><th>카테고리</th><th>그룹 변경</th></tr></thead><tbody>';
        groupItems.forEach(function(item) {
          var idx = items.indexOf(item);
          html += '<tr>' +
            '<td>' + item.sku + '</td>' +
            '<td>' + item.name + '</td>' +
            '<td>' + item.category + '</td>' +
            '<td>' + buildGroupSelect(item.group, true) + ' <button class="btn btn--primary btn--small" data-idx="' + idx + '" data-action="assign">배정</button></td>' +
            '</tr>';
        });
        html += '</tbody></table>';
      }

      html += '</div></div>';
    });
    container.innerHTML = html;

    // 그룹 내 배정 버튼
    container.querySelectorAll('[data-action="assign"]').forEach(function(btn) {
      btn.addEventListener('click', function() {
        var idx = parseInt(btn.getAttribute('data-idx'));
        var sel = btn.parentNode.querySelector('.sku-group-select');
        items[idx].group = sel.value;
        renderGroupedView();
        if (document.getElementById('view-all').style.display !== 'none') renderAllView();
        if (document.getElementById('view-ungrouped').style.display !== 'none') renderUngroupedView();
      });
    });

    // 그룹 편집 버튼
    container.querySelectorAll('[data-action="edit-group"]').forEach(function(btn) {
      btn.addEventListener('click', function() {
        openGroupModal(btn.getAttribute('data-group-id'));
      });
    });

    // 그룹 삭제 버튼
    container.querySelectorAll('[data-action="delete-group"]').forEach(function(btn) {
      btn.addEventListener('click', function() {
        var gid = btn.getAttribute('data-group-id');
        if (!confirm('그룹을 삭제하면 해당 그룹의 SKU가 미배정 상태로 변경됩니다. 계속하시겠습니까?')) return;
        items.forEach(function(item) { if (item.group === gid) item.group = ''; });
        groups = groups.filter(function(g) { return g.id !== gid; });
        renderGroupedView();
        renderAllView();
        renderUngroupedView();
      });
    });
  }

  function renderUngroupedView() {
    var tbody = document.getElementById('ungrouped-sku-tbody');
    var ungrouped = items.filter(function(i) { return !i.group; });
    var rows = '';
    ungrouped.forEach(function(item) {
      var idx = items.indexOf(item);
      rows += '<tr>' +
        '<td>' + item.sku + '</td>' +
        '<td>' + item.name + '</td>' +
        '<td>' + item.category + '</td>' +
        '<td>' + buildGroupSelect('', false) + ' <button class="btn btn--primary btn--small" data-idx="' + idx + '" data-action="assign">배정</button></td>' +
        '</tr>';
    });
    if (rows === '') rows = '<tr><td colspan="4" style="text-align:center;color:#888;">미배정 SKU가 없습니다.</td></tr>';
    tbody.innerHTML = rows;

    tbody.querySelectorAll('[data-action="assign"]').forEach(function(btn) {
      btn.addEventListener('click', function() {
        var idx = parseInt(btn.getAttribute('data-idx'));
        var sel = btn.parentNode.querySelector('.sku-group-select');
        if (!sel.value) { alert('배정할 그룹을 선택하세요.'); return; }
        items[idx].group = sel.value;
        renderUngroupedView();
        if (document.getElementById('view-all').style.display !== 'none') renderAllView();
        if (document.getElementById('view-grouped').style.display !== 'none') renderGroupedView();
      });
    });
  }

  // 탭 전환
  function setView(viewId) {
    ['view-all', 'view-grouped', 'view-ungrouped'].forEach(function(id) {
      document.getElementById(id).style.display = (id === viewId) ? '' : 'none';
    });
    ['btn-view-all', 'btn-view-grouped', 'btn-view-ungrouped'].forEach(function(id) {
      document.getElementById(id).classList.remove('active');
    });
    var btnMap = { 'view-all': 'btn-view-all', 'view-grouped': 'btn-view-grouped', 'view-ungrouped': 'btn-view-ungrouped' };
    document.getElementById(btnMap[viewId]).classList.add('active');

    if (viewId === 'view-all') renderAllView();
    if (viewId === 'view-grouped') renderGroupedView();
    if (viewId === 'view-ungrouped') renderUngroupedView();
  }

  document.getElementById('btn-view-all').addEventListener('click', function() { setView('view-all'); });
  document.getElementById('btn-view-grouped').addEventListener('click', function() { setView('view-grouped'); });
  document.getElementById('btn-view-ungrouped').addEventListener('click', function() { setView('view-ungrouped'); });

  // 그룹 추가/편집 모달
  var editingGroupId = null;

  function openGroupModal(groupId) {
    editingGroupId = groupId || null;
    var modal = document.getElementById('group-modal');
    var title = document.getElementById('modal-title');
    var nameInput = document.getElementById('modal-group-name');
    var descInput = document.getElementById('modal-group-desc');
    if (editingGroupId) {
      var g = groups.find(function(g) { return g.id === editingGroupId; });
      title.textContent = '그룹 편집';
      nameInput.value = g ? g.name : '';
      descInput.value = g ? (g.description || '') : '';
    } else {
      title.textContent = '새 그룹 추가';
      nameInput.value = '';
      descInput.value = '';
    }
    modal.style.display = 'flex';
    nameInput.focus();
  }

  document.getElementById('btn-add-group').addEventListener('click', function() {
    openGroupModal(null);
  });

  document.getElementById('modal-cancel').addEventListener('click', function() {
    document.getElementById('group-modal').style.display = 'none';
  });

  document.getElementById('modal-save').addEventListener('click', function() {
    var name = document.getElementById('modal-group-name').value.trim();
    var desc = document.getElementById('modal-group-desc').value.trim();
    if (!name) { alert('그룹명을 입력하세요.'); return; }

    if (editingGroupId) {
      var g = groups.find(function(g) { return g.id === editingGroupId; });
      if (g) { g.name = name; g.description = desc; }
    } else {
      var newId = 'group-' + Date.now();
      groups.push({ id: newId, name: name, description: desc });
    }

    document.getElementById('group-modal').style.display = 'none';
    renderAllView();
    if (document.getElementById('view-grouped').style.display !== 'none') renderGroupedView();
    if (document.getElementById('view-ungrouped').style.display !== 'none') renderUngroupedView();
  });

  // 초기 렌더링
  renderAllView();
})();
</script>
