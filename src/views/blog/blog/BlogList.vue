<template>
  <div>
    <!--搜索-->
    <el-row>
      <el-col :span="8">
        <!-- @input="selectPinYin(queryInfo.title)" -->
        <el-input placeholder="请输入标题"
                  v-model="queryInfo.title"
                  :clearable="true"
                  @clear="search"
                  @keyup.native.enter="search"
                  size="small"
                  style="min-width: 500px">
          <el-select v-model="queryInfo.categoryId"
                     slot="prepend"
                     placeholder="请选择分类"
                     :clearable="true"
                     @change="search"
                     style="width: 160px">
            <el-option :label="item.name"
                       :value="item.id"
                       v-for="item in categoryList"
                       :key="item.id"></el-option>
          </el-select>

          <el-button slot="append"
                     icon="el-icon-search"
                     @click="search"></el-button>
        </el-input>
      </el-col>
    </el-row>
    <!-- <p style="white-space: pre-wrap;"
       v-html="v"></p> -->
    <el-table :data="blogList">
      <el-table-column label="序号"
                       type="index"
                       width="50"></el-table-column>
      <el-table-column label="标题"
                       prop="title"
                       show-overflow-tooltip></el-table-column>
      <el-table-column label="分类"
                       prop="category.name"
                       width="150"></el-table-column>
      <el-table-column label="置顶"
                       width="80">
        <template v-slot="scope">
          <el-switch v-model="scope.row.top"
                     @change="blogTopChanged(scope.row)"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="推荐"
                       width="80">
        <template v-slot="scope">
          <el-switch v-model="scope.row.recommend"
                     @change="blogRecommendChanged(scope.row)"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="可见性"
                       width="100">
        <template v-slot="scope">
          <el-link icon="el-icon-edit"
                   :underline="false"
                   @click="editBlogVisibility(scope.row)">
            {{ scope.row.published ? (scope.row.password !== '' ? '密码保护' : '公开') : '私密' }}
          </el-link>
        </template>
      </el-table-column>
      <el-table-column label="创建时间"
                       width="170">
        <template v-slot="scope">{{ scope.row.createTime | dateFormat }}</template>
      </el-table-column>
      <el-table-column label="最近更新"
                       width="170">
        <template v-slot="scope">{{ scope.row.updateTime | dateFormat }}</template>
      </el-table-column>
      <el-table-column label="操作"
                       width="200">
        <template v-slot="scope">
          <el-button type="primary"
                     icon="el-icon-edit"
                     size="mini"
                     @click="goBlogEditPage(scope.row.id)">编辑</el-button>
          <el-popconfirm title="确定删除吗？"
                         icon="el-icon-delete"
                         iconColor="red"
                         @onConfirm="deleteBlogById(scope.row.id)">
            <el-button size="mini"
                       type="danger"
                       icon="el-icon-delete"
                       slot="reference">删除</el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!--分页-->
    <el-pagination @size-change="handleSizeChange"
                   @current-change="handleCurrentChange"
                   :current-page="queryInfo.pageNum"
                   :page-sizes="[10, 20, 30, 50]"
                   :page-size="queryInfo.pageSize"
                   :total="total"
                   layout="total, sizes, prev, pager, next, jumper"
                   background>
    </el-pagination>

    <!--编辑可见性状态对话框-->
    <el-dialog title="博客可见性"
               width="30%"
               :visible.sync="dialogVisible">
      <!--内容主体-->
      <el-form label-width="50px"
               @submit.native.prevent>
        <el-form-item>
          <el-radio-group v-model="radio">
            <el-radio :label="1">公开</el-radio>
            <el-radio :label="2">私密</el-radio>
            <el-radio :label="3">密码保护</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="密码"
                      v-if="radio===3">
          <el-input v-model="visForm.password"></el-input>
        </el-form-item>
        <el-form-item v-if="radio!==2">
          <el-row>
            <el-col :span="6">
              <el-switch v-model="visForm.appreciation"
                         active-text="赞赏"></el-switch>
            </el-col>
            <el-col :span="6">
              <el-switch v-model="visForm.recommend"
                         active-text="推荐"></el-switch>
            </el-col>
            <el-col :span="6">
              <el-switch v-model="visForm.commentEnabled"
                         active-text="评论"></el-switch>
            </el-col>
            <el-col :span="6">
              <el-switch v-model="visForm.top"
                         active-text="置顶"></el-switch>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
      <!--底部-->
      <span slot="footer">
        <el-button @click="dialogVisible=false">取 消</el-button>
        <el-button type="primary"
                   @click="saveVisibility">保存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import Breadcrumb from "@/components/Breadcrumb";
import { getDataByQuery, deleteBlogById, updateTop, updateRecommend, updateVisibility } from '@/api/blog'
import { createLogger } from 'vuex';
import { SimplePinYin } from "../../../components/PinYin/PinYin";

export default {
  name: "BlogList",
  components: { Breadcrumb },
  data () {
    return {
      queryInfo: {
        title: '',
        categoryId: null,
        pageNum: 1,
        pageSize: 10
      },
      v: "\n\n{\n   \"Java\":{\n      \"工作职责\":[\n         \"设计、开发、测试、维护Java应用程序\",\n         \"根据业务需求提供优秀的解决方案\",\n         \"参与系统集成、部署工作\"\n      ],\n      \"任职要求\":[\n         \"至少1年Java相关开发经验\",\n         \"熟悉SpringMVC、MyBatis等开源框架\",\n         \"熟悉数据库技术\",\n         \"具有良好的沟通能力和团队协作能力\",\n         \"熟悉常见的设计模式和调试工具\"\n      ],\n      \"备注\":\"我们注重员工的成长和个人价值的发掘和提升，提供具有竞争力的薪酬和福利待遇。\",\n      \"最需要的性格特质\":[\n         \"条理\", \"负责\", \"效率\", \"稳定\", \"冷静\"\n      ]\n   },\n   \"C++\":{\n      \"工作职责\":[\n         \"参与C++相关项目的需求分析、系统设计、编码\",\n         \"参与系统集成和调试工作\",\n         \"定位并解决产品问题\"\n      ],\n      \"任职要求\":[\n         \"至少1年C++开发经验\",\n         \"熟悉STL等常用的C++库\",\n         \"具备高效的代码阅读和调试能力\",\n         \"熟悉Linux或Windows环境下的开发\",\n         \"良好的团队合作和沟通能力\"\n      ],\n      \"备注\":\"我们提供有竞争力的薪资和福利，并且为C++开发人员提供广阔的职业发展和成长空间。\",\n      \"最需要的性格特质\":[\n         \"好奇\", \"谦恭\", \"条理\", \"稳定\", \"冷静\"\n      ]\n   },\n   \"PHP\":{\n      \"工作职责\":[\n         \"参与开发基于PHP的Web应用程序\",\n         \"解决项目存在的问题和bug\",\n         \"优化项目性能\",\n         \"与团队成员协作，保证项目进度和质量\"\n      ],\n      \"任职要求\":[\n         \"至少1年PHP开发经验，熟悉Laravel框架\",\n         \"掌握MySQL及相关优化方法\",\n         \"熟练运用队列处理、缓存等技术\",\n         \"具备扎实的计算机基础和良好的团队合作能力\"\n      ],\n      \"备注\":\"我们是一家创业公司，注重员工的成长和发展，并提供有竞争力的薪资和福利。\",\n      \"最需要的性格特质\":[\n         \"条理\", \"效率\", \"社交\", \"活力\", \"开朗\"\n      ]\n   },\n   \"数据挖掘\":{\n      \"工作职责\":[\n         \"负责大数据平台的数据清洗、处理、挖掘和分析\",\n         \"参与大规模数据架构设计和实现\",\n         \"对业务数据进行分析并提出改进建议\",\n         \"参与数据挖掘算法的研究和实现\"\n      ],\n      \"任职要求\":[\n         \"熟练掌握数据挖掘相关算法和技术，具有大数据分析经验\",\n         \"熟悉Hadoop、Spark等分布式计算平台和相关的开源技术\",\n         \"熟练掌握常用的机器学习和统计分析技术\",\n         \"良好的沟通和团队合作能力\"\n      ],\n      \"备注\":\"我们公司是一家关注员工技能提升和个人成长的企业，欢迎有志于从事数据挖掘领域的人加入我们。\",\n      \"最需要的性格特质\":[\n         \"好奇\", \"审美\", \"想象\", \"条理\", \"效率\"\n      ]\n   },\n   \"C\":{\n      \"工作职责\":[\n         \"负责嵌入式系统软件的设计、开发、调试和维护\",\n         \"解决项目中的技术难题\",\n         \"参与需求分析和系统设计\",\n         \"与团队成员进行合作和沟通，保证项目进展和质量\"\n      ],\n      \"任职要求\":[\n         \"掌握C语言编程，了解常见的数据结构和算法\",\n         \"掌握常用的嵌入式系统开发工具链和开发流程，如交叉编译、调试技巧等\",\n         \"有较好的软件开发能力，能独立完成项目的开发和调试工作\",\n         \"良好的团队合作精神和沟通能力\"\n      ],\n      \"备注\":\"作为一家从事嵌入式软件开发的企业，我们重视技术人才的培养和成长，提供有竞争力的薪酬和福利。\",\n      \"最需要的性格特质\":[\n         \"条理\", \"负责\", \"效率\", \"稳定\", \"冷静\"\n      ]\n   },\n   \"C#\":{\n      \"工作职责\":[\n         \"负责开发基于Microsoft .NET平台的应用系统\",\n         \"参与技术方案的设计和评估\",\n         \"推动项目的技术创新和最佳实践\",\n         \"保证项目进度和质量，并及时解决项目相关的问题和bug\"\n      ],\n      \"任职要求\":[\n         \"掌握Microsoft .NET及相关框架，熟悉C#语言\",\n         \"熟悉面向对象的思想、设计模式，具有良好的程序设计能力\",\n         \"熟悉SQL Server、Oracle等数据库的使用和优化\",\n         \"良好的团队合作和沟通能力\"\n      ],\n      \"备注\":\"我们的企业文化注重员工的培养和成长，提供有竞争力的薪资和福利待遇，同时强调团队合作和分享。\",\n      \"最需要的性格特质\":[\n         \"好奇\", \"想象\", \"条理\", \"负责\", \"冷静\"\n      ]\n   },\n   \".NET\":{\n      \"工作职责\":[\n         \"参与数字化转型项目的技术开发和推进\",\n         \"参与需求分析、系统设计和技术架构的制定\",\n         \"推动数字化转型项目的技术创新和最佳实践\",\n         \"保证项目进度和质量，并及时解决项目相关的问题和bug\"\n      ],\n      \"任职要求\":[\n         \"熟练掌握Microsoft .NET技术栈，熟悉ASP.NET、ADO.NET等开发框架\",\n         \"掌握常用的前端技术，如HTML、CSS、JavaScript等\",\n         \"熟悉各种常见的设计模式和开发模式\",\n         \"有较强的数据模型设计和数据库开发经验，掌握常见的SQL优化技术\",\n         \"良好的团队合作和沟通能力\"\n      ],\n      \"备注\":\"我们的数字化转型团队欢迎有志于从事数字化转型项目的技术人员加入，提供具有竞争力的薪酬和福利待遇。\",\n      \"最需要的性格特质\":[\n         \"审美\", \"条理\", \"负责\", \"冷静\", \"稳定\"\n      ]\n   }\n}",
      blogList: [],
      categoryList: [],
      total: 0,
      dialogVisible: false,
      blogId: 0,
      radio: 1,
      visForm: {
        appreciation: false,
        recommend: false,
        commentEnabled: false,
        top: false,
        published: false,
        password: '',
      },
      pinYinData: []
    }
  },
  created () {
    this.getData()
  },
  methods: {
    // 文字转拼音
    // selectPinYin (inputs, i) {
    //   const value = this.$pinyin(inputs, inputs)
    //   if (value) {
    //     this.queryInfo.title = inputs
    //     this.search()
    //   } else {
    //     this.queryInfo.title = inputs
    //     this.search()
    //   }
    // },

    // 拼音转文字 失败
    // selectPinYin (inputs, i) {
    //   SimplePinYin._pyvalue.forEach((item, index) => {
    //     let item1 = SimplePinYin._pyvalue[index]
    //     let item2 = SimplePinYin._pystr
    //     item1.push(item2[index])
    //     this.pinYinData.push(item1)
    //   })
    //   // 以拼音输入搜索
    //   if (inputs.charCodeAt() >= 32 && inputs.charCodeAt() <= 126) {
    //     const value = inputs.split('')
    //     const newValue = value.map(item => this.chinesChangPY(item)).join('')
    //     return newValue.indexOf(inputs.toLowerCase()) >= 0
    //     // 以中文搜索
    //   } else {
    //     return inputs
    //   }
    // },
    // chinesChangPY (item) {
    //   if (item.charCodeAt() < 32 || item.charCodeAt() > 126) {
    //     const py = this.pinYinData.filter(key => key[1].indexOf(item) >= 0)[0]
    //     return py[0]
    //   } else {
    //     return item
    //   }
    // },
    getData () {
      getDataByQuery(this.queryInfo).then(res => {
        this.blogList = res.data.blogs.list
        this.categoryList = res.data.categories
        this.total = res.data.blogs.total
      })
    },
    search () {
      this.queryInfo.pageNum = 1
      this.queryInfo.pageSize = 10
      this.getData()
    },
    //切换博客置顶状态
    blogTopChanged (row) {
      updateTop(row.id, row.top).then(res => {
        this.msgSuccess(res.msg);
      })
    },
    //切换博客推荐状态
    blogRecommendChanged (row) {
      updateRecommend(row.id, row.recommend).then(res => {
        this.msgSuccess(res.msg);
      })
    },
    //编辑博客可见性
    editBlogVisibility (row) {
      this.visForm = {
        appreciation: row.appreciation,
        recommend: row.recommend,
        commentEnabled: row.commentEnabled,
        top: row.top,
        published: row.published,
        password: row.password,
      }
      this.blogId = row.id
      this.radio = this.visForm.published ? (this.visForm.password !== '' ? 3 : 1) : 2
      this.dialogVisible = true
    },
    //修改博客可见性
    saveVisibility () {
      if (this.radio === 3 && (this.visForm.password === '' || this.visForm.password === null)) {
        return this.msgError("密码保护模式必须填写密码！")
      }
      if (this.radio === 2) {
        this.visForm.appreciation = false
        this.visForm.recommend = false
        this.visForm.commentEnabled = false
        this.visForm.top = false
        this.visForm.published = false
      } else {
        this.visForm.published = true
      }
      if (this.radio !== 3) {
        this.visForm.password = ''
      }
      updateVisibility(this.blogId, this.visForm).then(res => {
        this.msgSuccess(res.msg)
        this.getData()
        this.dialogVisible = false
      })
    },
    //监听 pageSize 改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pageSize = newSize
      this.getData()
    },
    //监听页码改变的事件
    handleCurrentChange (newPage) {
      this.queryInfo.pageNum = newPage
      this.getData()
    },
    goBlogEditPage (id) {
      this.$router.push(`/blog/edit/${id}`)
    },
    deleteBlogById (id) {
      this.$confirm('此操作将永久删除该博客<strong style="color: red">及其所有评论</strong>，是否删除?<br>建议将博客置为<strong style="color: red">私密</strong>状态！', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        dangerouslyUseHTMLString: true
      }).then(() => {
        deleteBlogById(id).then(res => {
          this.msgSuccess(res.msg)
          this.getData()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style scoped>
.el-button + span {
  margin-left: 10px;
}
</style>