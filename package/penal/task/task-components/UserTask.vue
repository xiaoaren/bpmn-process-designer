<template>
  <div style="margin-top: 16px">
    <el-form-item label="处理用户">
      <el-select v-model="userTaskForm.assignee" filterable reserve-keyword :loading="assigneeLoading" @change="updateElementTask('assignee')">
        <el-option v-for="item in selectAssigneeList" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="候选用户">
      <el-select v-model="userTaskForm.candidateUsers" multiple collapse-tags filterable reserve-keyword
        :loading="userListLoading" @change="updateElementTask('candidateUsers')">
        <el-option v-for="item in selectCandidateUserList" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="候选角色">
      <el-select v-model="userTaskForm.candidateGroups" multiple collapse-tags filterable reserve-keyword
        :loading="groupListLoading" @change="updateElementTask('candidateGroups')">
        <el-option v-for="item in selectCandidateGroupsList" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </el-form-item>
    <el-form-item label="到期时间">
      <el-input v-model="userTaskForm.dueDate" clearable @change="updateElementTask('dueDate')" />
    </el-form-item>
    <el-form-item label="跟踪时间">
      <el-input v-model="userTaskForm.followUpDate" clearable @change="updateElementTask('followUpDate')" />
    </el-form-item>
    <el-form-item label="优先级">
      <el-input v-model="userTaskForm.priority" clearable @change="updateElementTask('priority')" />
    </el-form-item>
  </div>
</template>

<script>
export default {
  name: "UserTask",
  props: {
    id: String,
    type: String
  },
  data() {
    return {
      defaultTaskForm: {
        assignee: "",
        candidateUsers: [],
        candidateGroups: [],
        dueDate: "",
        followUpDate: "",
        priority: ""
      },
      assigneeLoading: false,
      userListLoading: false,
      groupListLoading: false,
      userTaskForm: {},
      selectAssigneeList: [],
      defSelectAssigneeList: [],
      selectCandidateUserList: [],
      defSelectCandidateUserList: [],
      selectCandidateGroupsList: [],
      defSelectCandidateGroupsList: [],
    };
  },
  watch: {
    id: {
      immediate: true,
      handler() {
        this.bpmnElement = window.bpmnInstances.bpmnElement;
        this.$nextTick(() => this.resetTaskForm());
      }
    }
  },
  created() {
    window.FUN_UTILS.initUserTask(this);
    window.FUN_UTILS.userRemoteMethod('', (list) => {
      this.selectAssigneeList = list;
      this.defSelectAssigneeList = list;
      this.selectCandidateUserList = list;
      this.defSelectCandidateUserList = list;
    });
    window.FUN_UTILS.roleRemoteMethod('', (list) => {
      this.selectCandidateGroupsList = list;
      this.defSelectCandidateGroupsList = list;
    });
  },
  methods: {
    resetTaskForm() {
      for (let key in this.defaultTaskForm) {
        let value;
        if (key === "candidateUsers" || key === "candidateGroups") {
          value = this.bpmnElement?.businessObject[key] ? this.bpmnElement.businessObject[key].split(",") : [];
        } else {
          value = this.bpmnElement?.businessObject[key] || this.defaultTaskForm[key];
        }
        this.$set(this.userTaskForm, key, value);
      }
    },
    assigneeRemoteMethod(query) {
      if (query !== '') {
        this.assigneeLoading = true;
        window.FUN_UTILS.userRemoteMethod({name: query, appendIds:  11}, (options) => {
          this.assigneeLoading = false;
          this.selectAssigneeList = options;
        });
      } else {
        this.selectAssigneeList = this.defSelectAssigneeList;
      }
    },
    userListRemoteMethod(query) {
      if (query !== '') {
        this.userListLoading = true;
        window.FUN_UTILS.userRemoteMethod(query, (options) => {
          this.userListLoading = false;
          this.selectCandidateUserList = options;
        });
      } else {
        this.selectCandidateUserList = this.defSelectCandidateUserList;
      }
    },
    groupListRemoteMethod(query) {
      if (query !== '') {
        this.groupListLoading = true;
        window.FUN_UTILS.roleRemoteMethod(query, (options) => {
          this.groupListLoading = false;
          this.selectCandidateGroupsList = options;
        });
      } else {
        this.selectCandidateGroupsList = this.defSelectCandidateGroupsList;
      }
    },
    updateElementTask(key) {
      const taskAttr = Object.create(null);
      if (key === "candidateUsers" || key === "candidateGroups") {
        taskAttr[key] = this.userTaskForm[key] && this.userTaskForm[key].length ? this.userTaskForm[key].join() : null;
      } else {
        taskAttr[key] = this.userTaskForm[key] || null;
      }
      window.bpmnInstances.modeling.updateProperties(this.bpmnElement, taskAttr);
    }
  },
  beforeDestroy() {
    this.bpmnElement = null;
  }
};
</script>
