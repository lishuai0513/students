<template>
  <div>
    <Card class="mb-10">
      <Row>
        <Col span="12">
          <h3>管理员管理</h3>
        </Col>
        <Col span="12" class="text-right">
          <Button type="error" @click="myDelete">删除</Button>
        </Col>
      </Row>
    </Card>
    <Card>
    <Row style="margin-bottom: 10px;">
      <span>关键字：</span>
      <Input placeholder="请输入姓名查询" v-model.trim='searchText' style="width:200px;margin-right:20px;"></Input>
      <Button type="primary" @click='search'>搜索</Button>
    </Row>
        <Table size="large" @on-selection-change="selectOption" stripe ref="selection" :columns="tableCol" :data="tableData"></Table>
        <Page class='mt-10 text-right' :total="total" :current='currentPage' :page-size='pageSize' @on-change="changePage" @on-page-size-change="changeSize" show-sizer show-total />
    </Card>
    <!-- <Modal
      v-model="recordModal"
      title="操作"
      @on-cancel="closeRecordModal"
      width =60>
      <Table height="300" stripe :columns="recordCol" :data="recordData"></Table>
      <Button type="success" long @click="recordLoadMore">加载更多</Button>
      <div slot="footer">
          <Button type="default"  @click="closeRecordModal">关闭</Button>
      </div>
    </Modal> -->
  </div>
</template>
<script>
import { ManagerTableData, commentDel,recordTableData,editLandlordData} from '@/api/data'
export default {
  data () {
    return {
      // startTime: '', // 入住日期初始化
      // endTime: '',  // 退房日期初始化
      searchText: '', // 搜索内容初始化
      pageSize:10,
      currentPage:1,
      total:1,

//      recordModal:false,     //记录模态框
//      tableCurrentPage:1,    //住房记录当前页码
//      tableTotalPage:1,     //住房记录总页码
//      userId:'',      //住房记录用户id

      selectItemArr: [],    // 选中项目id
      tableCol: [
        {
          type: 'selection',
          width: 60,
          align: 'center'
        },
        {
          title: '姓名',
          key: 'name'
        },
        {
           title: '手机号',
           key: 'phone'
        },
        {
           title: '权限内容',
           key: 'roleName'
        },
        /*{
          title: '操作',
          key: 'examineStatus',
          width:100,
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'primary',
                  size: 'large'
                },
                on: {
                  click: () => {
                    this.userId = params.row.id;
                    this.tableInit();
                    this.recordModal = true;
                  }
                }
              }, '详情')
            ])
          }
        }*/
      ],
      tableData: [
        /*{
          createTime: '2018-01-03',
          idNumber: '111111111111111111',
          thisHourse: '2',
          phone: '18738790000',
          name: '用户1',
          id: 1
          // record: "详情",
        }*/
      ],
      /*recordCol:[
        {
          title: '管理员名称',
          key: 'userneme'
        },
        {
          title: '房屋地址',
          key: 'positionAddr'
        },
        {
          title: '房屋名称',
          key: 'name'
        },
        {
          title: '入住时间',
          key: 'intime'
        },
        {
          title: '退房时间',
          key: 'outtime'
        },
      ],*/
      recordData:[
      ]
    }
  },
  methods: {
    async pageInit(){
      ManagerTableData(this.pageSize,this.currentPage,this.searchText).then(res => {
        if(res.data.success){
          this.total = res.data.result.pageBounds.totalSize;
          this.tableData = res.data.result.pageBounds.pageList;
        }else if(res.data.errorMsg){
          this.$Message.warning({
            content:res.data.errorMsg
          })
        }
      })
    },
    /*async tableInit(){
      recordTableData(this.userId,this.tableCurrentPage).then(res => {
        console.log("+++++"+JSON.stringify(res));
        if(res.data.success){
          this.tableCurrentPage = res.data.result.data.currectPage;
          this.tableTotalPage = res.data.result.data.totalPage;
          var dataList = res.data.result.data.pageList;
          var recordO = {};
          for(let item of dataList) {
            recordO = {};
            recordO.id = item.id;
            recordO.intime = item.intime;
            recordO.outtime = item.outtime;
            recordO.area = item.lock.longAddress;
            recordO.positionAddr = item.lock.positionAddr;
            recordO.name = item.lock.name;
            this.recordData.push(recordO);
          }
        }else if(res.data.errorMsg){
          this.$Message.warning({
            content:res.data.errorMsg
          })
        }
      })
    },*/
    changePage(page){
      this.currentPage = page;
      this.pageInit();
    },
    changeSize(size){
      this.pageSize = size;
      this.currentPage = 1;
      this.pageInit();
    },
    // 选中项
    selectOption (selection, row) {
      this.selectItemArr = []
      for (let item of selection) {
        this.selectItemArr.push(item.id)
      }
    },
    // 删除
    myDelete () {
      var selectItemArr = this.selectItemArr
      if(selectItemArr.length<1){
        this.$Message.warning({
          content:'请选择项目！'
        })
        return;
      }
      commentDel('/manager/delManager', selectItemArr).then(res => {
        if(res.data.success){
          this.selectItemArr=[];
          this.pageInit();
          this.$Message.success({
            content:'删除成功！'
          });
        }else if(res.data.errorMsg){
          this.$Message.warning({
            content:res.data.errorMsg
          })
        }

        // this.tableData = res.data.result.data.pageList;
      })
    },
    // 入住日期
    myStartTime (newTime) {
      this.startTime = newTime
    },
    // 退房日期
    myEndTime (newTime) {
      this.endTime = newTime
    },
    // 搜索
    search () {
      this.currentPage = 1;
      this.pageInit();
    },
   /* recordLoadMore(){
      if(this.tableCurrentPage === this.tableTotalPage){
        this.$Message.warning({
          content:'暂无更多数据'
        })
        return;
      }
      this.tableCurrentPage += 1;
      this.tableInit();
    },*/
    closeRecordModal(){
      this.recordModal = false;
      this.tableCurrentPage = 1;
      this.recordData = [];
    }

  },
  mounted () {
    // 初始化
    ManagerTableData(10, 1).then(res => {
      if(res.data.success){
        console.log("+++++"+JSON.stringify(res))
        this.total = res.data.result.pageBounds.totalSize;
        this.tableData = res.data.result.pageBounds.pageList;
      }else if(res.data.errorMsg){
        this.$Message.warning({
          content:res.data.errorMsg
        })
      }
    })
  }
}
</script>
