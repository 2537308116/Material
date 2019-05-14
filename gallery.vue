<template>
 <div class='gallery'>
      <div class="gallerys_bg" v-if="show">
           <div class="bg_input">
               <p>{{text}}<i class="el-icon-close" @click="cancel" style="float:right;margin-top: 14px;"></i></p>
               <el-input v-model="groupName" ></el-input>
               <button class="bg_btn" @click="addClass">确认</button>
               <button class="bg_btn1" @click="cancel">取消</button>
           </div>
      </div>
      <p>素材库<i class="el-icon-close" @click="Close" style="float:right"></i></p>
      <div class="btn">
           <!-- <el-button type="danger" icon="el-icon-delete">删除</el-button> -->
           <el-button type="primary" style="float:right" @click="updataSc">上传<i class="el-icon-upload el-icon--right"></i></el-button>
           <input type="file" @change="previewImage(this)" id="previewImg" style="display: none;">
      </div>
      <div class="classification">
          <ul>
              <li v-for="(itme,index) in list" :key="index" :class="{'active':ind == itme.groupId}" @click="btnClick(itme.groupId)"> 
                  <i @click="addlist(itme.groupId)" class="el-icon-edit" style="float:left;margin-top: 13px;margin-left: 5px;"></i>
                  {{itme.groupName}}
                  <i @click="Classdelete(itme.groupId)" class="el-icon-circle-close" style="float:right;margin-top: 13px;margin-right: 5px;"></i>
              </li>
          </ul>
          <p @click="addlist">新增分组</p>
      </div>
      <div class="center">
           <!-- <video src="https://zlhr-wgapp.oss-cn-shenzhen.aliyuncs.com/20190424/f4c2d8a9170f466a871b854145b2d598.mp4" style="width: 100px;height: 100px;object-fit: fill"></video> -->
           <ul>
               <li v-for="(itme,index) in resourcesList" :key="index"  @click="resourClick(itme.imagesId,itme.domainName,itme.imagesUrl,itme.fileType)">
                   <img v-if="itme.fileType == 1" :src="itme.domainName+itme.imagesUrl" alt="">
                   <video v-if="itme.fileType == 2" :src="itme.domainName+itme.imagesUrl" style="object-fit: fill"></video>
                   <div class="select_mask" v-if="resour.indexOf(itme.imagesId) != -1">
                        <img src="../../assets/chose.png">
                   </div>
               </li>
           </ul>
      </div>
      <div class="gallbtn">
           <el-button @click="query" type="primary">确定</el-button>
           <el-button @click="Close">取消</el-button>
      </div>
 </div>
</template>

<script>
import {Addclassification,ClassList,ClassDelete,Flieupload,Resources,Classdetails,Classupdate} from '../../api/api'
export default {
    data() {
        return {
            groupName:'',
            show:false,
            list:[],
            ind:"1",
            resourcesList:[],
            resour:[],
            text:"请输入新分组名称",
            groupId:'',
            getdata:[],
        };
    },
    methods: {
      addlist(groupId){ //编辑添加分类
            if(typeof groupId == "number"){
                this.text = "修改分组名称" 
                this.groupId = groupId
                Classdetails(groupId).then(res =>{
                    if(res.data.code == 0){
                        this.groupName = res.data.data.groupName
                    }
                })
            }else{
                this.text = "请输入新分组名称"  
                this.groupId = ''
            } 
         this.show = true
      },
      query(){
          if(this.getdata.length == 0){
                this.$message({
                    message: '请选择素材',
                    type: 'error'
                });
          }else{
                this.$emit('getData',this.getdata);
                this.$store.state.show=false
                this.$message({
                    message: '选择成功',
                    type: 'success'
                });
          }
      },
      //关闭弹框
      Close(){
         this.$store.state.show=false
      },
      //选择资源
      resourClick(index,domainName,imagesUrl,fileType){
          if(this.resour.indexOf(index)  ==-1){
              this.resour.push(index)
              this.getdata.push({imagesId:index,imagesUrl:domainName+imagesUrl,fileType:fileType})
          }else{
              let num = this.resour.indexOf(index);
              if (num > -1) {
                 this.resour.splice(num, 1);
              }
              for(let i = 0; i<this.getdata.length; i++){
                  if(this.getdata[i].imagesId == index){
                     this.getdata.splice(i, 1);
                  }
              }
          }
      },
      //上传素材
      updataSc(){
        document.querySelector("#previewImg").click();
      },
      previewImage($event){  
         let that = this
         var reader =new FileReader();
             var file =document.querySelector("#previewImg").files;
              reader.readAsDataURL(file[0]);
              var formdata = new FormData()
              if(file[0].type == 'image/jpeg' || file[0].type == 'image/png' || file[0].type == 'image/jpg'){
                 formdata.append('fileType',1)
              }else if(file[0].type == 'video/mp4'){
                 formdata.append('fileType',2)
              }
             formdata.append('file',file[0])
             formdata.append('groupId',that.ind)
              Flieupload(formdata).then(res =>{
                  if(res.data.code == 0){
                        this.$message({
                            message: '上传成功',
                            type: 'success'
                        });
                     this.resources(that.ind)
                  }
              })
              reader.onload=function(){
         }
     },
      btnClick(index){
        this.ind = index
        this.resources(index)
      },
      cancel(){  //关闭输入框
         this.show = false
      },
      Classdelete(groupId){  //删除分类
            this.$confirm('确定删除该分组?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).then(() => {
                    ClassDelete([groupId]).then(res =>{
                        if(res.data.code == 0){
                            this.$message({
                                message: '删除成功',
                                type: 'success'
                            });
                            this.classList()
                        }else{
                            this.$message({
                                message: res.data.msg,
                                type: 'success'
                            });
                        }
                    })
                }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
      },
      addClass(){
          if(this.groupName == ''){
              this.$message({
                  message: '请输入分组名称',
                  type: 'error'
              });
          }else{
                let promse ={
                    groupName:this.groupName
                }
               if(this.groupId){
                    promse.groupId = this.groupId
                    Classupdate(promse).then(res =>{
                        if(res.data.code == 0){
                            this.$message({
                                message: '修改成功',
                                type: 'success'
                            });
                            this.show = false
                            this.classList()
                        }else{
                            this.$message({
                                message: '修改失败',
                                type: 'error'
                            });
                        }
                    })
               }else{
                    Addclassification(promse).then(res =>{
                        if(res.data.code == 0){
                            this.$message({
                                message: '添加成功',
                                type: 'success'
                            });
                            this.show = false
                            this.classList()
                        }else{
                            this.$message({
                                message: '修改失败',
                                type: 'error'
                            });
                        }
                    })
               }
          }
      },
      classList(){
        ClassList().then(res =>{
            if(res.data.code == 0){
                this.list = res.data.data
            }
        })
      },
      resources(id){
          let prmose ={
              groupId:id
          }
          Resources(prmose).then(res =>{
              if(res.data.code == 0){
                 this.resourcesList = res.data.data
              }
          })
      }
    },
    mounted() {
       this.classList()
       this.resources()
    },
}
</script>
<style scoped lang="scss">
.active{
    background-color: rgba(48, 145, 242, 0.1) !important;
    color: #0e90d2 !important;
}
.borderClass{

}
.gallery{
    position: fixed;
    width: 800px;
    height: 500px;
    background-color: #fff;
    top: 150px;
    left: 50%;
    margin-left: -400px;
    box-shadow: 1px 1px 50px rgba(0, 0, 0, .3);
    z-index: 99;
    .gallerys_bg{
      z-index: 991;
      position: fixed;
      width: 800px;
      height: 500px;
      top: 150px;
      left: 50%;
      margin-left: -400px;
      background-color: #00000042;
      .bg_input{
          width: 270px;
          height: 160px;
        //   background-color: white;
          background: url('../../assets/20190424140749.png');
          position: fixed;
          top: 300px;
          left: 50%;
          margin-left: -140px;
          z-index: 999;
          p{
              margin: 0;
              font-size: 14px;
              color: #333;
              height: 40px;
              line-height: 40px;
              padding-left: 20px;
              background-color: #F8F8F8;
              border-bottom: 1px solid #eee;
          }
          .el-input{
              width: 84%;
              margin-left: 8%;
              margin-top: 20px;
          }
          button{
              margin-top: 25px;
              margin-left: 20px;
              cursor: pointer; 
          }
          .bg_btn{
              border-color: #1E9FFF;
              background-color: #1E9FFF;
              color: #fff;
              margin-left: 130px;
          }
      }
    }
    p{
        padding: 0px 20px;
        font-size: 16px;
        cursor: pointer; 
    }
    .btn{
        height: 40px;
        padding-left: 160px;
        padding-right: 50px;
        .el-button--danger{
            height: 40px;
            line-height: 0;
        }
    }
    .classification{
        width: 150px;
        height: 380px;
        float: left;
        clear:both;
        ul{
            margin: 0;
            padding: 0;
            overflow: hidden;
            overflow-y: scroll;
            height: 310px;
        }
        li{
            list-style: none;
            cursor: pointer; 
            background: #fff;
            border-radius: 6px;
            width: 130px;
            height: 40px;
            line-height: 40px;
            text-align: center;
            margin: 2px 0;
            .el-icon-circle-close{
                display: none;
            }
            .el-icon-edit{
                display: none;
            }
        }
        // li:first-child{
        //     background-color: rgba(48, 145, 242, 0.1);
        //     color: #0e90d2;
        // }
        li:hover {
            background-color: rgba(48, 145, 242, 0.1);
            color: #0e90d2;
            .el-icon-circle-close{
                display: block;
            }
            .el-icon-edit{
                display: block;
            }
        }
        p{
            font-size: 14px;
            text-align: center;
            color: #0e90d2;
            cursor: pointer; 
        }
    }
    .center{
        width: 600px;
        height: 320px;
        float: left;
        margin-left: 10px;
        margin-top: 20px;
        overflow: hidden;
        overflow-y: scroll;
        // background-color: red;
        ul{
            margin: 0;
            padding: 0;
            li{
                float: left;
                height: 160px;
                width: 136px;
                position: relative;
                // background-color: red;
                list-style: none;
                margin: 2px 2px;
                text-align: center;
                line-height: 160px;
                border: 1px solid rgba(0, 0, 0, 0.05);
                border-radius: 6px;
                img{
                    width:126px ;
                    height: 155px;
                    margin-top: 3px;
                }
                video{
                    width:126px ;
                    height: 155px;
                    margin-top: 3px;
                }
                .select_mask{
                    // display: none;
                    height: 160px;
                    width: 136px;
                    position: absolute;
                    top: 0;
                    background: rgba(0, 0, 0, 0.5);
                    img{
                        position: absolute;
                        top: 50px;
                        left: 35px;
                        width: 65px;
                        height: 50px;
                    }
                }
            }
            li:hover{
                border: 1px solid #16bce2;
            }
        }
    }
    .gallbtn{
        margin-top: 360px;
        text-align: right;
        button{
            margin-right: 30px;
        }
    }
    
}
</style>