<template>
  <div>
    <div style="padding: 10px 0;display: flex;justify-content: space-between">
      <el-input style="width: 300px" placeholder="请输入文件名" v-model="input">
        <el-button slot="append" icon="el-icon-search" @click="search"></el-button>
      </el-input>
      <el-button icon="el-icon-plus" @click="upload"  style="margin-left: 5px" v-if="permissions==='1'"></el-button>
    </div>
    <el-card v-loading="loading" element-loading-text="文件下载中.....">
<!--      <xd-file-list-preview :show-close="showClose" :list="list" @remove="handleRemoveClick"></xd-file-list-preview>-->
      <div v-if="!total">
        <el-empty :image-size="260"></el-empty>
      </div>
      <div v-else>
        <el-row>
          <el-col :span="6" v-for="(item,index) in list">
            <div class="xd-file-list__item">
              <div class="-icon">
                <img src="../../assets/jpg.png" alt="icon" style="width:100%;height:100%" v-if="item.type.toLowerCase()==='jpg'||item.type.toLowerCase()==='jpeg'||item.type.toLowerCase()==='png'">
                <img src="../../assets/pdf.png" alt="icon" style="width:100%;height:100%" v-else-if="item.type.toLowerCase()==='pdf'">
                <img src="../../assets/ppt.png" alt="icon" style="width:100%;height:100%" v-else-if="item.type.toLowerCase()==='pptx'">
                <img src="../../assets/word.png" alt="icon" style="width:100%;height:100%" v-else-if="item.type.toLowerCase()==='doc'||item.type.toLowerCase()==='docx'">
                <img src="../../assets/xls.png" alt="icon" style="width:100%;height:100%" v-else-if="item.type.toLowerCase()==='xls'||item.type.toLowerCase()==='xlsx'">
                <img src="../../assets/unkown.png" alt="icon" style="width:100%;height:100%" v-else>
              </div>
              <div class="-text">
                <el-tooltip class="item" effect="dark" :content="item.name" placement="bottom" :open-delay="openDelay">
                  <div class="tmp">{{item.name}}</div>
                </el-tooltip>
                <div style="display: flex">
                  <div class="-link" @click="preview(item)">预览</div>
                  <div class="-link" style="margin-left: 6px" @click="download(item.url)">下载</div>
                </div>
              </div>
              <i class="fileIconfont iconyduicuowushixin" @click.stop="handleRemoveClick(item.url)" v-if="permissions==='1'"></i>
            </div>
          </el-col>
        </el-row>
      </div>
      <div style="padding: 10px 0">
        <el-pagination
            @current-change="handleCurrentChange"
            :current-page.sync="pageNum"
            :page-size="pageSize"
            layout="prev, pager, next, jumper"
            :total="total">
        </el-pagination>
      </div>
    </el-card>
    <el-dialog title="上传资源" :visible.sync="dialogFormVisible" width="30%" :before-close="close">
      <el-form label-width="110px" size="small" :model="fileForm" ref="fileForm" v-loading="loading2" element-loading-text="文件上传中.....">
        <el-form-item label="文件命名：" prop="fileName">
          <el-input placeholder="请输入文件名" v-model="fileForm.fileName"></el-input>
        </el-form-item>
        <el-form-item label="文件下载：" prop="download">
          <el-switch v-model="fileForm.private" active-color="#C0CCDA" inactive-color="#13ce66" active-value="0" inactive-value="1"></el-switch>
          <span style="font-size: 12px">（开启后不允许用户在线下载，默认开启）</span>
        </el-form-item>
        <el-upload
            class="upload-demo"
            drag
            ref="upload"
            multiple
            action=""
            accept=".pdf,.jpg,.jpeg,.png,.docx,.pptx,.xls,.xlsx"
            :with-credentials="true"
            :http-request="httpRequest"
            :on-exceed="handleExceed"
            :limit="limit"
            :file-list="sendFile"
            :data="fileData"
            :auto-upload="false"
            style="text-align: center">
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">
            <em>点击上传</em>
            (支持上传pdf，png，jpeg，jpg，docx，pptx，xls，xlsx等文件)
          </div>
        </el-upload>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="uploadFile">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="预览" :visible.sync="pdf.pdfDialog" width="70%" :before-close="pdfClose" :destroy-on-close="destroy">
      <el-button-group>
        <el-button type="primary" icon="el-icon-arrow-left" size="mini" @click="prePage">上一页</el-button>
        <el-button type="primary" size="mini" @click="nextPage">下一页<i class="el-icon-arrow-right el-icon--right"></i></el-button>
      </el-button-group>
      <div style="marginTop: 10px; color: #409EFF">{{ pdf.page }} / {{ pdf.pageTotalNum }}</div>
      <pdf :page="pdf.page" :src="pdf.url" @progress="pdf.loadedRatio = $event" @num-pages="pdf.pageTotalNum=$event"></pdf>
    </el-dialog>
    <el-dialog title="预览" :visible.sync="excel.dialog" width="90%" :before-close="excelClose" :destroy-on-close="destroy">
      <base-excel ref="excelChild"></base-excel>
    </el-dialog>
    <el-dialog title="预览" :visible.sync="word.dialog" width="90%" :before-close="wordClose" :destroy-on-close="destroy">
      <base-word ref="wordChild"></base-word>
    </el-dialog>
    <el-dialog title="预览" :visible.sync="pptx.dialog" width="80%" :before-close="pptxClose" :destroy-on-close="destroy">
      <pptx ref="pptxChild" v-if="pptx.dialog"></pptx>
    </el-dialog>
    <el-image-viewer v-if="showViewer" :on-close="closeViewer" :url-list="[url]"/><!--图片预览-->
  </div>
</template>

<script>
import {mapGetters} from "vuex";
import {iconData} from "@/assets/contact";
import pdf from 'vue-pdf'
import baseExcel from './excel'
import baseWord from './word'
import pptx from "@/views/file/pptx";
import ElImageViewer from 'element-ui/packages/image/src/image-viewer'
import axios from 'axios'
export default {
  name: "File",
  components:{
    pdf,
    baseExcel,
    baseWord,
    pptx,
    ElImageViewer
  },
  data() {
    return {
      showViewer: false, // 图片显示查看器
      url:'',// 图片显示url
      pdf:{
        url: '',
        page: 1,
        pageTotalNum: 1,
        loadedRatio: 0,
        pdfDialog:false,
      },
      excel:{dialog:false},
      word:{dialog:false},
      pptx:{dialog:false},
      showClose: true,
      limit: 2,
      input:'',
      input2:'',
      dialogFormVisible:false,
      fileForm:{fileName:'',private:'1'},//1:默认开启下载，0：关闭下载
      sendFile: [],
      list: [
        // {url: 'http://152.136.122.135:8848/file/show?url=./file/1/hustoj%E6%96%87%E6%A1%A3%E5%A4%A7%E5%85%A8.pdf',name:'2334'},
      ],
      pageNum:1,
      pageSize:16,
      fileData:null,
      query:{},
      openDelay:500,
      destroy:true,
      loading:false,
      loading2:false,
      header:{'token':localStorage.getItem('token')}
    }
  },
  computed:{
    ...mapGetters({
      fileList:'fileStore/fileList',
      total:'fileStore/total',
      permissions:'loginStore/permissions',
    }),
  },
  created() {
    this.load(1,'')
  },
  methods:{
    load(page,title){
      this.query.id = parseInt(this.$route.params.courseId)
      this.query.page = page
      this.query.title = title
      let _this = this
      this.$store.dispatch('fileStore/getFileResources',this.query).then(res=>{
        let list = [];
        if(res.code===200){
          _this.fileList.forEach(function (item, index) {
            list.push({url:item.url,name:item.name,download:item.private==='1'?true:false,type:item.type})
          });
          _this.list = list
        }
      })
    },
    httpRequest(param) {
      this.loading2 = true
      let fileObj = param.file // 相当于input里取得的files
      const size = fileObj.size/1024/1024
      if(size>50){
        this.$notify.warning({
          title:'警告',
          message:'大小必须小于50M'
        })
        return;
      }
      let fd = new FormData()// FormData 对象
      fd.append('file', fileObj)// 文件对象
      this.fileData = {courseid:this.$route.params.courseId,title:this.fileForm.fileName,file:fd,private:this.fileForm.private}
      this.$store.dispatch('fileStore/postFileUploadOne',this.fileData).then(res=>{
        if(res.code === 200){
          this.close()
          this.load(1,'')
          this.$message.success(res.msg)
        }else{
          this.$message.warning(res.msg)
        }
        this.loading2 = false
      })
    },
    uploadFile(){
      this.$refs.upload.submit();
    },
    handleExceed(files, fileList) {
      this.$message.warning(`当前限制选择 1 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`);
    },
    upload(){
      this.dialogFormVisible = true
    },
    close(){
      this.dialogFormVisible = false
      if(this.$refs.upload){
        this.$refs.upload.clearFiles()//清除上传的文件
      }
      this.fileForm = {fileName:'',private:'1'}
      this.sendFile = []
    },
    handleCurrentChange(pageNum) {
      this.pageNum = pageNum
      this.load(pageNum,this.input2)
    },
    search(){
      this.input2 = this.input
      this.pageNum = 1
      this.load(this.pageNum,this.input2)
    },
    preview(item){//预览文件
      if(item.type.toLowerCase() === 'pdf'){
        this.pdf.pdfDialog = true
        let loadingTask = pdf.createLoadingTask({
          url:'http://192.168.0.123:8848/file/download?url='+encodeURIComponent(item.url),
          httpHeaders:{'token':localStorage.getItem('token')}
        })
        this.pdf.url = loadingTask
      }else if(item.type.toLowerCase() === 'xls'||item.type.toLowerCase() === 'xlsx'){
        this.excel.dialog =true
        this.$nextTick(()=>{
          this.$refs.excelChild.excel('http://192.168.0.123:8848/file/download?url='+encodeURIComponent(item.url));
        })
      }else if(item.type.toLowerCase() === 'docx'||item.type.toLowerCase() === 'doc'){
        this.word.dialog = true
        this.$nextTick(()=>{
          this.$refs.wordChild.word('http://192.168.0.123:8848/file/download?url='+encodeURIComponent(item.url));
        })
      }else if(item.type.toLowerCase()==='jpg'||item.type.toLowerCase()==='jpeg'||item.type.toLowerCase()==='png'){
        let url = this.imgSrc('http://192.168.0.123:8848/file/download?url='+encodeURIComponent(item.url))
        url.then(res =>{///////////////////获取promise对象中的value
          this.onPreview(res)
        } )
      }else if(item.type.toLowerCase()==='pptx'){
        this.pptx.dialog = true
        this.$nextTick(()=>{
          this.$refs.pptxChild.pptx('http://192.168.0.123:8848/file/download?url='+encodeURIComponent(item.url));
        })
      }
    },
    async imgSrc(url) {
      let that = this;
      let urlsrc = "";  //通过一个参数 从下面接收

      // 通过图片地址获取图片，从新获取图片
      var config = {
        method: "get",
        responseType: "arraybuffer",
        url: url,
        headers: {token:localStorage.getItem('token')}
      };
      // 重新获取请求，获取的是base64位的图片
      await axios(config).then((response) => {
        //因为从这里return老是报错
        urlsrc = "data:image/png;base64," +
            btoa(new Uint8Array(response.data).reduce((data, byte) => data + String.fromCharCode(byte) + "", ""));
      });
      return urlsrc;
    },
    handleRemoveClick(url) {//删除文件
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$store.dispatch('fileStore/postFileDelete',{id:parseInt(this.$route.params.courseId),url:url}).then(res=>{
          this.$message.success('文件删除成功！')
          this.load(1,'')
        })
      }).catch(() => {
        this.$message({type: 'info', message: '已取消删除'});
      });
    },
    prePage() {//pdf预览翻页
      let page = this.pdf.page
      page = page > 1 ? page - 1 : this.pdf.pageTotalNum
      this.pdf.page = page
    },
    nextPage() {//pdf预览翻页
      let page = this.pdf.page
      page = page < this.pdf.pageTotalNum ? page + 1 : 1
      this.pdf.page = page
    },
    pdfClose(){
      this.pdf = {url: '',page: 1, pageTotalNum: 1, loadedRatio: 0, pdfDialog:false,}
    },
    excelClose(){
      this.excel.dialog = false
      // this.$nextTick(()=>{
      //   this.$refs.excelChild.excelClear();
      // })
    },
    wordClose(){
      this.word.dialog = false
      // this.$nextTick(()=>{
      //   this.$refs.wordChild.wordClear();
      // })
    },
    pptxClose(){
      this.pptx.dialog = false
      // this.$nextTick(()=>{
      //   this.$refs.pptxChild.close();
      // })
    },
    onPreview(url) {
      this.showViewer = true
      this.url = url
    },
    // 关闭查看器
    closeViewer() {
      this.showViewer = false
    },
    download(url){
      //window.open('http://152.136.122.135:8848/file/download?url='+encodeURIComponent(url))
      this.loading = true
      this.$store.dispatch('fileStore/getFileDownload',{url:url}).then(res=>{
        const link=document.createElement('a');
        try{
          let blob =  res
          let _fileName = url.substr(url.lastIndexOf('/')+1,url.length)
          link.style.display='none';
          // 兼容不同浏览器的URL对象
          const urls = window.URL || window.webkitURL || window.moxURL;
          link.href=urls.createObjectURL(blob);
          link.download = _fileName;
          link.click();
          window.URL.revokeObjectURL(urls);
          this.loading = false
        }catch (e) {
          this.$message.error('下载文件出错,'+e)
        }
      }).catch(()=>{
        this.$message.error('下载文件出错！')
      })
      // axios.get('http://152.136.122.135:8848/file/download',{
      //   responseType: "blob",
      //   params:{url:url}
      // }).then(({data})=> {
      //   const link=document.createElement('a');
      //   try{
      //     let blob =  data
      //     let _fileName = url.substr(url.lastIndexOf('/')+1,url.length)
      //     link.style.display='none';
      //     // 兼容不同浏览器的URL对象
      //     const urls = window.URL || window.webkitURL || window.moxURL;
      //     link.href=urls.createObjectURL(blob);
      //     link.download = _fileName;
      //     link.click();
      //     window.URL.revokeObjectURL(urls);
      //   }catch (e) {
      //     this.$message.error('下载文件出错,'+e)
      //   }
      // }).catch(()=>{
      //   this.$message.error('下载文件出错')
      // })
    }
  }
}
</script>

<style scoped>
/deep/ .el-upload .el-upload-dragger{
  width: 100%;
}

.bodys{
  padding-top: 20px;
  color: #333;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-wrap: wrap;
}

.xd-file-list__item{
  position: relative;
  padding: 12px;
  box-sizing: border-box;
  height: 68px;
  background: #f6f7fa;
  border: 1px solid #cacad1;
  border-radius: 5px;
  width: 294px;
  margin: 0 20px 20px 0;
  display: flex;
  justify-content: center;
  align-items: center;
  align-content: flex-start;
  text-align: left;
  flex-wrap: nowrap;
}
.xd-file-list__item .-icon{
  width: 50px;
  height: 50px;
  margin-right: 10px;
  background: #fff;

}
.xd-file-list__item .-text{
  flex: 1;
}
.xd-file-list__item .-link {
   height: 20px;
   font-size: 14px;
   font-family: PingFangSC, PingFangSC-Regular;
   color: #4285f4;
   line-height: 20px;
   cursor: pointer;
 }
.xd-file-list__item .-link:hover {
   text-decoration: underline;
 }


.xd-file-list__item i{
  margin-top: 3px;
  display: inline-block;
}
.xd-file-list__item i.iconyduicuowushixin {
  cursor: pointer;
  display: none;
  position: absolute;
  top: 0px;
  right: 0px;
  width: 20px;
  height: 20px;
  font-size: 21px;
  color: #999;
}
.xd-file-list__item:hover i.iconyduicuowushixin {
  display: block;
}
.tmp{
  width: 205px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}
</style>
