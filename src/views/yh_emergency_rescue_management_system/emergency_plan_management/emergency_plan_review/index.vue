<template>
  <a-card :bordered="false">

    <!--     查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="30">
          <a-col :xl="7" :lg="10" :md="12" :sm="24">
            <a-form-item label="预案名称">
              <j-input placeholder="请输入名称模糊查询" v-model="queryParam.emergencyPlanName"></j-input>
            </a-form-item>
          </a-col>

          <a-col ::xl="7" :lg="10" :md="12" :sm="24">
            <a-form-item label='预案种类'>
              <!--              <j-input placeholder="请输入预案种类" v-model="queryParam.emergencyPlanCategory"></j-input>-->
              <a-select placeholder="请选择预案种类" v-model="queryParam.emergencyPlanCategory">
                <a-select-option v-for="item in inputData.emergencyPlanCategory" :value="item">
                  {{item}}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :xl="7" :lg="10" :md="12" :sm="24">
            <a-form-item label="发布状态">
              <a-select placeholder="请选择发布状态" v-model="queryParam.isReleased">
                <a-select-option v-for="item in inputData.isReleased" :value="item">
                  {{item}}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <template v-if="toggleSearchStatus">
            <a-col :xl="7" :lg="10" :md="12"  :sm="24">
              <a-form-item label="">
                <a-form-item label="发布时间">
                  <a-range-picker v-model="queryParam.applyTime"
                                  format="YYYY-MM-DD"
                                  :placeholder="['开始时间', '结束时间']"
                                  @change="onIssueDateChange" />
                </a-form-item>
              </a-form-item>
            </a-col>

            <a-col :xl="7" :lg="10" :md="12" :sm="24">
              <a-form-item label="审批状态">
                <a-select placeholder="请选择审批状态" v-model="queryParam.applicationState">
                  <a-select-option v-for="item in inputData.applicationState" :value="item">
                    {{item}}
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

          </template>


          <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </a-col>
          </span>
        </a-row>
      </a-form>
    </div>

    <!--     操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!--      <a-button type="primary" icon="plus" @click="jump">创建单据</a-button>-->
      <!--      <a-button type="primary" icon="plus" @click="onetomany">一对多</a-button>-->
      <a-button type="primary" icon="download" >导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <!--      <j-super-query :fieldList="fieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->

      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!--     table区域-begin-->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
          selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
          <a-divider type="vertical" />
        </span>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        :rowKey="record=>record.emergencyPlanId"
        :columns="defColumns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">


        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.emergencyPlanId)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>

              <a-menu-item>
                  <a @click="()=>handleCheak(record)">详情</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <modal ref="modalForm" @ok="modalFormOk"></modal>
    <check-modal ref="checkModal"></check-modal>
    <!-- 一对多表单区域 -->
    <!--    <JeecgDemoTabsModal ref="jeecgDemoTabsModal" @ok="modalFormOk"></JeecgDemoTabsModal>-->

  </a-card>
</template>

<script>
import Modal from './childComponents/Modal'
import { copyObj } from 'codemirror/src/util/misc'
import CheckModal from './childComponents/CheckModal'
export default {
  name: 'index.vue',
  data(){
    return{
      selectedRowKeys:[],
      /* 查询条件-请不要在queryParam中声明非字符串值的属性 */
      queryParam: {},
      /* 数据源 */
      inputData: {
        emergencyPlanCategory:["综合应急预案" , "专项应急预案"] ,
        applicationState:["编辑中","审批中","通过","未通过","被退回"],
        isReleased:['已发布','未发布']
      },
      dataSource:[
        {
          emergencyPlanId:"12323",
          emergencyPlanName:"测试应急预案01",
          version:"1.0",
          emergencyPlanCategory:"综合应急预案",
          specifiedTime:"2020-9-10",
          updateTime:"2020-9-10",
          departName:"测试部",
          fillPerson:"管理员张",
          applyTime:"2020-9-10",
          uploadFileName:"",
          applicationState:"通过",
          isReleased:"已发布",
        },
        {
          emergencyPlanId:"12423",
          emergencyPlanName:"测试应急预案01",
          version:"1.0",
          emergencyPlanCategory:"综合应急预案",
          specifiedTime:"2020-9-10",
          updateTime:"2020-9-10",
          departName:"测试部",
          fillPerson:"管理员张",
          applyTime:"2020-9-10",
          uploadFileName:"",
          applicationState:"通过",
          isReleased:"已发布",
        },
        {
          emergencyPlanId:"15323",
          emergencyPlanName:"测试应急预案01",
          version:"1.0",
          emergencyPlanCategory:"综合应急预案",
          specifiedTime:"2020-9-10",
          updateTime:"2020-9-10",
          departName:"测试部",
          fillPerson:"管理员张",
          applyTime:"2020-9-10",
          uploadFileName:"",
          applicationState:"通过",
          isReleased:"已发布",
        },

      ],
      /* 分页参数 */
      ipagination:{
        current: 1,
        pageSize: 10,
        pageSizeOptions: ['10', '20', '30'],
        showTotal: (total, range) => {
          return range[0] + "-" + range[1] + " 共" + total + "条"
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },

      // 默认列
      defColumns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: "center",
          customRender: function (t, r, index) {
            return parseInt(index) + 1;
          }
        },
        {
          title: '应急预案名称',
          align: "center",
          dataIndex: 'emergencyPlanName'
        },
        {
          title: '版本号',
          align: "center",
          dataIndex: 'version'
        },
        {
          title: '预案类别',
          align: "center",
          dataIndex: 'emergencyPlanCategory'
        },
        {
          title: '制定时间',
          align: 'center',
          dataIndex: 'specifiedTime'
        },
        {
          title: '申请时间',
          align: "center",
          dataIndex: 'applyTime'
        },
        {
          title: '申请状态',
          align: "center",
          dataIndex: 'applicationState'
        },
        {
          title: '发布状态',
          align: "center",
          dataIndex: 'isReleased'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: "center",
          scopedSlots: {
            filterDropdown: 'filterDropdown',
            filterIcon: 'filterIcon',
            customRender: 'action'
          },
        }
      ],

      settingColumns:[],

      toggleSearchStatus:false
    }
  },
  components:{
    Modal,
    CheckModal
  },
  methods:{
    //搜索方法
    searchQuery(){

    },

    loadData(){

    },

    searchReset(){

    },

    onIssueDateChange: function (value, dateString) {
      console.log(dateString[0],dateString[1]);
      this.queryParam.birthday_begin=dateString[0];
      this.queryParam.birthday_end=dateString[1];
    },

    handleToggleSearch(){
      this.toggleSearchStatus = !this.toggleSearchStatus;
    },

    onClearSelected() {
      this.selectedRowKeys = [];
      this.selectionRows = [];
    },

    onColSettingsChange(){

    },

    handleEdit(record){
      this.$refs.modalForm.edit(record);
      this.$refs.modalForm.title = "编辑应急预案";
      this.$refs.modalForm.method = "edit";
      this.$refs.modalForm.disableSubmit = false;

    },

    handleAdd(){
      this.$refs.modalForm.add();
      this.$refs.modalForm.title = "新增应急预案";
      this.$refs.modalForm.method = "add";
      this.$refs.modalForm.disableSubmit = false;
    },

    handleDelete(id){
      const dataSource = [...this.dataSource];
      this.dataSource = dataSource.filter(item => item.emergencyPlanId !== id);
    },

    handleCheak(record){
      this.$refs.checkModal.check(record);
      this.$refs.checkModal.title = "查看应急预案";
      this.$refs.checkModal.confirmLoading = false;
    },

    batchDel(){

    },

    // 加载数据
    modalFormOk(data){
      if(data.method === "add"){
        this.dataSource.push(data.modelData);
      }
      else if(data.method === "edit") {
        const dataSource = [...this.dataSource];
        const target = dataSource.find(item => item.id === data.modelData.id);
        if (target) {
          copyObj(data.modelData,target);
          this.dataSource = dataSource;
        }
      }
    },

    handleTableChange(pagination, filters, sorter) {
      // //分页、排序、筛选变化时触发
      // //TODO 筛选
      // console.log(pagination)
      // if (Object.keys(sorter).length > 0) {
      //   this.isorter.column = sorter.field;
      //   this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
      // }
      // this.ipagination = pagination;
      // this.loadData();
    },

    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys;
      this.selectionRows = selectionRows;
    },


  }

}
</script>

<style scoped>

</style>