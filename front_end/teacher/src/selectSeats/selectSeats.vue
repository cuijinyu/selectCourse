<template>
  <div>
    <el-row :gutter="5">
      <el-col :span="12" v-for="c in classes"  type="flex">
        <el-card class="box-card" style="margin-top:20px;">
          <div slot="header" class="clearfix">
            <span>{{c.name}}</span>
            <el-button style="float: right; padding: 3px 0" type="text" :key="c.no" v-on:click="searchClass(c.no,c.stage)">查询此课程的选课状况</el-button>
          </div>
          <div>
            <p>教室:{{c.classroom}}</p>
            <p>授课教师:{{c.teacher}}</p>
            <p>开课时间:{{c.time}}</p>
            <p>期数:{{c.stage}}</p>
            <p>开始选课时间:{{c.startDateTime}}</p>
            <p>结束选课时间:{{c.endDateTime}}</p>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <!--座位显示页面-->
    <el-dialog title="选择座位" :visible.sync="chooseSeat">
      <el-row :gutter="10">
        <div class="seat" v-for="a in seats">
          <button class="innerSeat" v-if="a.seatAble"></button>
          <button class="disableSeat" v-if="!a.seatAble" disabled></button>
        </div>
      </el-row>
    </el-dialog>
  </div>
</template>

<script>

    let config = require("../../config/config");
    const ip = config.ip;
    let moment = require("moment");
    let $ = require("jquery");
    let _ = require("underscore");

  /**
   * 以下是seat类的定义
   */
    class Seat{
      constructor(x,y){
        this.x = x;
        this.y = y;
        this.seatAble = true;
        this.hasBeenChoose = false;
      }
      getPosition(){
        return{
          x:this.x,
          y:this.y
        }
      }
      setSeatAble(state){
        this.seatAble = false;
      }
      setHasBeenChoose(){
        this.hasBeenChoose = true;
      }
      getSeatAble(){
        return this.seatAble
      }
    }


    function getHasBeenChoosedSeats(courseNo,stage){
      /**
       *    获取到已经被选择的座位
       * */
      return new Promise((resolve,reject)=>{
        $.post(`http://${ip}/course/seatStatus`,{
          courseNo:courseNo,
          stage:stage
        },function (response) {
          if(response.code == 1001){
            resolve(response.data);
          }else{
            reject(response.msg);
          }
        })
      })
    }


    let map = [];//一个临时转储变量
    export default {
        name: "select-seats",
        data(){
          return{
            chooseSeat:false,
            beenChooseClass:"",
            beenChooseSeat:"",
            seats:[],//保存座位的数组
            classes:[
              // {
              //   no:1,
              //   name:'数据结构和算法',
              //   classroom:'B103',
              //   teacher:'李明',
              //   time:'2017-1-18',
              //   stage:'一期',
              //   startSelectDate:'2017-1-9',
              //   endSelectDate:'2017-1-12'
              // },{
              //   no:2,
              //   name:'数据结构和算法',
              //   classroom:'B103',
              //   teacher:'李明',
              //   time:'2017-1-18',
              //   stage:'一期',
              //   startSelectDate:'2017-1-9',
              //   endSelectDate:'2017-1-12'
              // },{
              //   no:3,
              //   name:'数据结构和算法',
              //   classroom:'B103',
              //   teacher:'李明',
              //   time:'2017-1-18',
              //   stage:'一期',
              //   startSelectDate:'2017-1-9',
              //   endSelectDate:'2017-1-12'
              // },{
              //   no:4,
              //   name:'数据结构和算法',
              //   classroom:'B103',
              //   teacher:'李明',
              //   time:'2017-1-18',
              //   stage:'一期',
              //   startSelectDate:'2017-1-9',
              //   endSelectDate:'2017-1-12'
              // },{
              //   no:5,
              //   name:'数据结构和算法',
              //   classroom:'B103',
              //   teacher:'李明',
              //   time:'2017-1-18',
              //   stage:'一期',
              //   startSelectDate:'2017-1-9',
              //   endSelectDate:'2017-1-12'
              // }
            ]
          }
        },
      methods:{
        /**
         *  获取课程列表
         *  @returns {null}
         */
        getClassLists() {
          let self = this;
          let result;
          $.post(`http://${ip}/course/allInfo`,{
            power:4
          },function (response) {
            result = response.data;
            console.log(result)
            result.forEach(function (ele) {
              ele.startDateTime = moment(ele.startDateTime).format("YYYY/MM/D,HH:mm:ss")
              ele.endDateTime = moment(ele.endDateTime).format("YYYY/MM/D,HH:mm:ss")
            })
            self.classes = result;
          })
        },
        /**
         *
         * @param no 课程编号
         * @param stage 课程期数
         * @returns {null}
         */
        searchClass(no,stage){
          this.buildMap();
          console.log(no);
          let self = this;
          getHasBeenChoosedSeats(no,stage)
            .then(function (row) {
              row.forEach(function (ele) {
                self.seats[ele].setSeatAble();
              })
            }).then(function () {
            self.beenChooseClass = no;
            self.beenChooseStage = stage;
            self.chooseSeat = true;
          }).catch(function (e) {
            self.$message.error("获取座位信息失败");
          })
        },

        /**
         *    构建一个全新的地图
          * */
        buildMap() {
          for(let i =0 ;i<352;i++){
            this.seats[i].seatAble = true;
          }
          this.seats[0].setSeatAble();
          this.seats[20].setSeatAble();
          this.seats[21].setSeatAble();
          this .seats[22*2-1].setSeatAble()
          this.seats[1*22].setSeatAble();
          this.seats[11*22].setSeatAble();
          this.seats[11*22+1].setSeatAble();
          this.seats[12*22].setSeatAble();
          this.seats[12*22+2].setSeatAble();
          this.seats[12*22+1].setSeatAble();
          this.seats[13*22].setSeatAble();
          this.seats[13*22+1].setSeatAble();
          this.seats[13*22+2].setSeatAble();
          this.seats[13*22+3].setSeatAble();
          this.seats[14*22].setSeatAble();
          this.seats[14*22+1].setSeatAble();
          this.seats[14*22+2].setSeatAble();
          this.seats[14*22+3].setSeatAble();
          this.seats[14*22+4].setSeatAble();
          this.seats[15*22].setSeatAble();
          this.seats[15*22+1].setSeatAble();
          this.seats[15*22+2].setSeatAble();
          this.seats[15*22+3].setSeatAble();
          this.seats[15*22+4].setSeatAble();
          this.seats[15*22+5].setSeatAble();
        }
      },
      beforeMount:function () {

        /**
         * 在组件挂载前加载课程列表
         */
        if(map.length > 200){
          this.seats = map;
        }else{
          for(let i = 0;i<16;i++){
            for(let j = 0;j<22;j++)
              map.push(new Seat(i,j))
          }
          for(let i = 0;i<16;i++){
            this.seats = map;
          }
        }
        this.getClassLists();
      }
    }
</script>

<style scoped>
  .hasBeenChoose{
    width:80% ;
    height: 90%;
    border:1px solid rgba(0,0,0,0.5);
    margin: 0 auto;
    background-color: lightgrey;
    -webkit-border-radius: 3px;
    -moz-border-radius: 3px;
    border-radius: 3px;
  }
  .disableSeat{
    width:80% ;
    height: 90%;
    border:1px solid rgba(0,0,0,0.2);
    margin: 0 auto;
    background-color:lightgrey;
    -webkit-border-radius: 3px;
    -moz-border-radius: 3px;
    border-radius: 3px;
  }
  .innerSeat:hover{
    transition: background-color 0.3s;
    background-color: rgb(64,158,225);
  }
  .seat{
    width:4.5%;
    /*padding:3px;*/
    /*margin: 5px;*/
    height: 30px;
    /*border:1px solid rgba(0,0,0,0.5);*/
    background:rgba(0,0,0,0);
    display: inline-block;
  }
  .innerSeat{
    width:80% ;
    height: 90%;
    border:1px solid rgba(0,0,0,0.5);
    margin: 0 auto;
    background-color: white;
    -webkit-border-radius: 3px;
    -moz-border-radius: 3px;
    border-radius: 3px;
  }
</style>
