<template>
  <div class="content">
    <div class="course-box">
      <h1 class="course-name" style="text-align: center; font-size: 28px;">{{courseInfo.course_name}}</h1>
      <h3 style="margin-left: 40px">简介：</h3>
      <p class="p">{{courseInfo.course_intro}}</p>
      <h3 class="course-item">授课教师：</h3>
      <p class="p">{{courseInfo.teacher_name}}</p>
      <h3 class="course-item">上课时间：</h3>
      <p
        style="margin-left: 60px; margin-top: 10px"
        v-for="(time,index) in courseInfo.time_slot"
        :key="index"
      >周{{time.week_day}} &ensp;第{{time.period}}节课 &ensp;{{time.start_time}}-{{time.end_time}}</p>
      <h3 class="course-item">课室：</h3>
      <p
        style="margin-top: 5px; margin-left: 60px; margin-bottom: 15px"
      >{{courseInfo.building}}-{{courseInfo.room}}</p>
    </div>
    <hr v-if="!isAdmin" class="hr" color="#ffd700">
    <!-- 管理员 -->
    <el-button v-if="isAdmin" type="primary" icon="el-icon-edit" @click="setCourse">编辑</el-button>
    <el-button v-if="isAdmin" type="danger" icon="el-icon-delete" @click="handleDelete">删除</el-button>

    <!-- 非管理员 -->
    <div v-if="!isAdmin" class="course-box" align="center">
      <el-card class="box-card" shadow="never" body-style="padding-left: 40px; padding-top: 20px">
        <div slot="header" class="clearfix" style="text-align: center">
          <span style="font-style: unset; font-size: 18px">通知</span>
          <!-- 教师操作 -->
          <el-button v-if="isTeacher" type="text" @click="addFormVisible = true" class="release">发布</el-button>
          <!-- 发布公告弹框 -->
          <el-dialog title="发布公告" :visible.sync="addFormVisible">
            <el-form :model="form">
              <el-form-item label="公告标题" label-width="120px">
                <el-input v-model="form.title" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="公告内容" label-width="120px">
                <el-input v-model="form.content" type="textarea" :rows="5" autocomplete="off"></el-input>
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button type="primary" @click="handleAddNotice">确认发布</el-button>
            </div>
          </el-dialog>
        </div>
        <div v-for="notice in notices" :key="notice.notice_id" class="text item">
            <img  src="../../assets/i6.png" style="height:18px; width: 18px;"/>
          <span style="margin-left: 10px">{{notice.title}}</span>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapState } from "vuex";

export default {
  data() {
    return {
      addFormVisible: false,
      form: {
        action: "post",
        sec_id: this.$route.params.course,
        title: "",
        content: ""
      }
    };
  },
  computed: {
    ...mapGetters("login", {
      isStudent: "isStudent",
      isTeacher: "isTeacher",
      isAdmin: "isAdmin"
    }),
    ...mapState({
      courseInfo: state => state.course.course,
      notices: state => state.notice.noticeList
    })
  },
  created() {
    this.$store.dispatch("course/fetchCourse", this.$route.params.course);
    this.$store.dispatch("notice/fetchNotice", this.$route.params.course);
  },
  beforeRouteUpdate(to, from, next) {
    this.$store.dispatch("course/fetchCourse", to.params.course);
    this.$store.dispatch("notice/fetchNotice", to.params.course);
    next();
  },

  methods: {
    handleAddNotice() {
      // 真正路径 '/notice'
      this.$http("post", "/admin/course", this.form).then(res => {
        if (res.data.status == 0) {
          this.$alert("发布成功", "消息", {
            confirmButtonText: "确定",
            beforeClose: (action, instance, done) => {
              this.$router.go("0");
          }
         }) // setTimeout(() => {}, 1000);
        } else {
          this.$alert(res.data.error_msg, "发布失败", {
            confirmButtonText: "确定"
          });
        }
      });
    },
    handleDelete() {
      this.$confirm(
        '是否确认删除"' + this.courseInfo.course_name + '"这门课程?',
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
          this.$http("post", "/admin/course", {
            action: "delete",
            section_id: this.$route.params.course
          }).then(res => {
            if (res.data.status == 0) {
              this.$message({type: "success", message: "删除成功!"});
              setTimeout(() => { this.$router.push('/coursemgt')}, 1000)
            } else {
              this.$message({
                type: "info",
                message: "删除失败"
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    setCourse() {
      this.$router.push({
        name: "setCourse",
        params: {
          operation: 'edit'
        }
      });
    }
  }
};
</script>

<style scoped>
.course-box {
  text-align: left;
  padding-left: 0px;
  margin: 10px;
}
.course-name {
  margin-bottom: 20px;
}
.course-item{
  margin-top: 20px;
  margin-left: 40px;
}
.hr {
  margin-left: 50px;
  margin-right: 50px;
  height:1px;
}
.p {
  margin-right: 20px;
  margin-top: 5px;
  margin-left: 60px;
}

/* 通知 */
.text {
  font-size: 14px;
  text-align: left;
}

/*修改*/
.item {
  margin-bottom: 10px;
  padding-top: 10px;
  padding-bottom: 10px;
  padding-left: 20px;
  margin-right: 60px;
}
.item:hover {
  background: aliceblue;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}

.box-card {
  border: 0;
  width: 100%;
  height: 100%;
}

.release {
  padding-left: 15px;
  padding-right: 15px;
  background: cornflowerblue;
  color: #fff;
  font-size: 12px;
  float: right;
}
</style>
