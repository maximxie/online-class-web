<template>
  <div>
    <el-card>
      <div>
        <span style="font-size: 15px;font-weight: bold">标题:</span>
        <el-input v-model="form.title" placeholder="请输入作业标题："></el-input>
      </div>
      <div style="margin-top: 20px">
        <span style="font-size: 15px;font-weight: bold">题目编号:</span>
        <el-input v-model="problem" placeholder="Example:1000,1001,1002" style="margin-bottom: 20px" @blur="inputBlur"></el-input>
        <div v-for="(item,index) in this.problem_title">
          <span>{{problem_list[index]}}:</span>
          <span>{{ item }}</span>
        </div>
      </div>
      <div style="margin-top: 20px">
        <span style="font-size: 15px;font-weight: bold">作业描述:</span>
      </div>
      <tinymce></tinymce><!-------------------------tinymce---------------------------------->
      <div style="display: flex;margin-top: 40px">
        <div>
          <span style="font-size: 15px;font-weight: bold">公开:</span>
          <el-select v-model="form.private" placeholder="请选择">
            <el-option
                v-for="item in options2"
                :key="item.value"
                :label="item.label"
                :value="item.value">
            </el-option>
          </el-select>
        </div>
        <div style="margin-left: 10px">
          <span style="font-size: 15px;font-weight: bold">密码:</span>
          <el-input v-model="form.password" size="mini" style="width: auto"></el-input>
        </div>
      </div>

      <div style="margin-top: 20px">
        <span style="font-size: 15px;font-weight: bold">语言:</span>
        <el-select v-model="lang2" multiple placeholder="请选择">
          <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
          </el-option>
        </el-select>
      </div>

    </el-card>
    <div style="margin-top: 10px">
      <el-button type="primary" @click="promote">确定</el-button>
      <el-button type="primary" @click="back">取消</el-button>
    </div>



    <el-dialog title="编辑作业" :visible.sync="dialogFormVisible" width="40%" :before-close="close">
      <el-form label-width="90px" size="small" :model="form" ref="form" :rules="rules">
        <el-form-item label="发布时间:" prop="start_time">
          <el-date-picker v-model="form.start_time" type="datetime" placeholder="选择日期时间" value-format="yyyy-MM-dd HH:mm:ss" :disabled="disabled2"></el-date-picker>
        </el-form-item>
        <el-form-item label="截止时间:" prop="end_time">
          <el-date-picker v-model="form.end_time" type="datetime" placeholder="选择日期时间" value-format="yyyy-MM-dd HH:mm:ss"></el-date-picker>
        </el-form-item>
        <el-form-item label="发布设置:">
          <el-radio v-model="radio" label="1" @change="closeDialog">统一发布</el-radio>
          <el-radio v-model="radio" label="2" @change="openDialog">分班发布</el-radio>
        </el-form-item>
        <el-form-item v-if="false" prop="class_id"></el-form-item>
        <el-transfer
            v-model="yesData"
            :props="{key: 'class_id',label: 'class_name'}"
            :titles="['全选', '已选择分班']"
            @change="handleChange"
            :data="noData"
            v-if="display"
        ></el-transfer>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="addShiXun" :disabled="disabled">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {mapGetters} from "vuex";
import tinymce from "@/components/tinymce";
import {lang} from "@/assets/data";
import moment from "moment";
export default {
  name: "shixun-homework-edit",
  components:{
    tinymce
  },
  computed:{
    ...mapGetters({
      problemTitle:'shixunStore/problemTitle',
      classAll:'classStore/classAll',
      editInfo:'shixunStore/editInfo'
    })
  },
  created() {
    this.load()
  },
  beforeDestroy () {
    window.tinymce.remove(`#${this.ID}`) //销毁
  },
  data() {
    return {
      form:{
        title:'', problem_id:[], private:0, password:'', langmask:0, start_time:'', end_time:'', description:'', class_id:[],contest_id:0
      },
      problem:'',
      problem_list:[],
      problem_title:[],
      nums: [],
      dialogFormVisible: false,
      radio: '1',
      Data:[],
      yesData: [],
      noData: [],
      display: false,
      lang1:[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20],
      lang2:[],
      options:lang,
      options2:[{value: 0, label: '公开'},{value: 1, label: '私有'}],
      disabled:false,
      disabled2:true,
      rules:{
        start_time:[
          {required: true, message: '请选择开始时间', trigger: 'blur'},
        ],
        end_time:[
          {required: true, message: '请选择结束时间', trigger: 'blur'},
        ]
      }
    }
  },
  methods: {
    load(){
      this.$store.dispatch('shixunStore/getTrainingEditInfo',{id:parseInt(this.$route.params.contestId)}).then(res=>{
        if(res.code === 200){
          this.form.contest_id = parseInt(this.$route.params.contestId)
          this.form.title = this.editInfo.title
          this.problem = JSON.stringify(this.editInfo.problem_id).substring(1,JSON.stringify(this.editInfo.problem_id).length-1)
          this.form.private = this.editInfo.private
          this.form.password = this.editInfo.password
          this.form.start_time = new Date(this.editInfo.start_time)
          this.form.end_time = new Date(this.editInfo.end_time)
          let lang_count = 21,lang2 = [],i = 0
          let langs = (~(parseInt(this.editInfo.langmask)))&((1<<lang_count)-1)
          this.lang1.forEach(function (item, index) {
            if(langs&(1<<index)){
              lang2[i++] = item
            }
          });
          this.lang2 = lang2
          this.inputBlur()
        }
      })
    },
    back(){
      this.$router.push({name:'shixun-homework-list'})
    },
    promote(){
      this.dialogFormVisible = true
      this.$store.dispatch('classStore/getClassAll',{id:parseInt(this.$route.params.courseId)}).then(res=>{//获取课堂下的所有班级
        if(this.classAll.length===0){
          this.disabled = true
          this.$message.warning('所在课堂未创建班级！')
        }else{
          let IntClassId=[],editClassId = this.editInfo.class_id,noData2=[],yesData2=[]
          this.classAll.forEach(function (item, index) {
            IntClassId[index] = parseInt(item.class_id)
            noData2.push({class_name:item.class_name,class_id:parseInt(item.class_id)})
            if(editClassId.indexOf(parseInt(item.class_id))!==-1){
              yesData2.push(parseInt(item.class_id))
            }
          });
          this.Data = IntClassId
          if(this.editInfo.class_id.length>=this.Data.length){
            this.radio = '1'
            this.display = false
            this.noData = noData2
          }else{
            this.radio = '2'
            this.display = true
            this.yesData = yesData2
            this.noData = noData2
          }
        }
      })
    },
    openDialog(){
      this.display = true
    },
    closeDialog(){
      this.display = false
    },
    handleChange(value, direction, movedKeys) {
      if(direction === "right") {
        //console.log(this.yesData)
      }
      if(direction === "left") {}
    },
    inputBlur(){
      this.problem_list = this.problem.split(',')
      //console.log(this.problem_list)
      this.$store.dispatch('shixunStore/getProblemTitle',{problem_id:this.problem_list}).then(res=>{
        if(res.code === -4){
          this.$message.warning(res.msg)
        }
        if(res.code === -3){
          this.$message.warning(res.msg)
          return ;
        }
        let problem=[],len = this.problemTitle.length
        this.problem_title = this.problemTitle
        this.problem_list.forEach(function (item, index) {
          if(index >= len) return;
          problem[index] = parseInt(item)
        });
        this.form.problem_id = problem
      })
    },
    addShiXun(){
      if(this.radio === '1'){
        this.form.class_id = this.Data
      }else{
        if(this.yesData.length===0){
          this.$message.error('请选择作业发布的班级!')
          return ;
        }
        this.form.class_id = this.yesData
      }
      let lang_count = 21
      let langmask = 0
      this.lang2.forEach(function (item, index) {
        langmask+=1<<item
      });
      langmask = ((1<<lang_count)-1)&(~langmask)
      this.form.langmask = langmask
      this.form.description = localStorage.getItem('tinymceContent')
      localStorage.removeItem("tinymceContent");
      this.$refs['form'].validate((valid) => {
        if(valid){
          let date1 = new Date(this.form.start_time)
          let date2 = new Date(this.form.end_time)
          if(date1<date2){
            if(this.form.title===''||this.problem_title.length===0){
              this.$message.warning('请输入作业标题或题目编号！')
              return;
            }
            //console.log(this.form)
            this.form.start_time = moment(this.form.start_time).format('YYYY-MM-DD HH:mm:ss')
            this.form.end_time = moment(this.form.end_time).format('YYYY-MM-DD HH:mm:ss')
            console.log(this.form)
            this.$store.dispatch('shixunStore/postTrainingEdit',this.form).then(res=>{
              if(res.code === 200){
                this.back()
                this.$message.success('修改实训作业成功!')
              }else{
                this.$message.warning(res.msg)
              }
            })
          }else{
            this.$message.warning('请输入合法的时间段！(开始时间应早于结束时间)')
          }
        }
      });
    },
    close(){
      this.dialogFormVisible = false
      this.$nextTick(() => {
        this.$refs["form"].resetFields();
      });
    },
  }
}
</script>

<style scoped>

</style>
