<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="员工编号">
              <j-input placeholder="请输入名称模糊查询" v-model="queryParam.employeeId"></j-input>
            </a-form-item>
          </a-col>

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="证书编号">
              <j-input placeholder="请输入名称模糊查询" v-model="queryParam.certNum"></j-input>
            </a-form-item>
          </a-col>

          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="签发日期">
                <a-range-picker v-model="queryParam.issueDate"
                                format="YYYY-MM-DD"
                                :placeholder="['开始时间', '结束时间']"
                                @change="onBirthdayChange" />
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

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
<!--      <a-button type="primary" icon="plus" @click="jump">创建单据</a-button>-->
<!--      <a-button type="primary" icon="plus" @click="onetomany">一对多</a-button>-->
<!--      <a-button type="primary" icon="download" @click="handleExportXls('单表示例')">导出</a-button>-->
<!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
<!--        <a-button type="primary" icon="import">导入</a-button>-->
<!--      </a-upload>-->
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="fieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>

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

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
          selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync" />刷新</a>
          <a-divider type="vertical" />
          <a-popover title="自定义列" trigger="click" placement="leftBottom">
            <template slot="content">
              <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
                <a-row style="width: 400px">
                  <template v-for="(item,index) in defColumns">
                    <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                        <a-col :span="12"><a-checkbox :value="item.dataIndex"><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox></a-col>
                    </template>
                  </template>
                </a-row>
              </a-checkbox-group>
            </template>
            <a><a-icon type="setting" />设置</a>
          </a-popover>
        </span>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <div slot="filterDropdown">
          <a-card>
<!--            表头第一行-->
            <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
              <a-row style="width: 400px">
                <template v-for="(item,index) in defColumns">
                  <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                    <a-col :span="12"><a-checkbox :value="item.dataIndex"><j-ellipsis :value="item.title" :length="10"></j-ellipsis></a-checkbox></a-col>
                  </template>
                </template>
              </a-row>
            </a-checkbox-group>
          </a-card>
        </div>
        <a-icon slot="filterIcon" type='setting' :style="{ fontSize:'16px',color:  '#108ee9' }" />

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
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
// import JeecgDemoModal from '@/views/jeecg/modules/JeecgDemoModal';
import JSuperQuery from '@/components/jeecg/JSuperQuery.vue';
import Modal from './childComponents/Model'
import CheckModal from './childComponents/CheckModal';
import JInput from '@/components/jeecg/JInput.vue';
// import JeecgDemoTabsModal from './modules/JeecgDemoTabsModal'
import {initDictOptions, filterDictText,filterDictTextByCache} from '@/components/dict/JDictSelectUtil'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import Vue from 'vue'
import { filterObj } from '@/utils/util';

//高级查询modal需要参数
const superQueryFieldList=[
  // {
  //   type: "string",
  //   value: "name",
  //   text: "用户名"
  // }
    ]
export default {
  name: "HealthCertManage",
  mixins:[JeecgListMixin],
  components: {
    Modal,
    JSuperQuery,
    JInput,
    CheckModal
  },
  data() {
    return {
      description: '单表示例列表',
      //字典数组缓存
      sexDictOptions: [],
      importExcelUrl:`${window._CONFIG['domianURL']}/test/jeecgDemo/importExcel`,
      //表头
      columns:[],
      //列设置
      settingColumns:[],
      //列定义
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
          title: '员工编号',
          align: "center",
          dataIndex: 'employeeId'
        },
        {
          title: '证书编号',
          align: "center",
          dataIndex: 'certNum'
        },
        {
          title: '部门',
          align: "center",
          dataIndex: 'apartment'
        },
        // {
        //   title: '性别',
        //   align: "center",
        //   dataIndex: 'sex',
        //   customRender: (text) => {
        //     //字典值替换通用方法
        //     return filterDictTextByCache('sex', text);
        //   }
        // },
        {
          title: '签发日期',
          align: "center",
          dataIndex: 'issueDate'
        },
        {
          title: '有效期至',
          align: "center",
          dataIndex: 'validity'
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
      // url: {
      //   //请求列表数据的地址
      //   list: "/wzh/yhHealthCert/list",
      //   delete: "/wzh/yhHealthCert/delete",
      //   deleteBatch: "/wzh/yhHealthCert/deleteBatch",
      //   exportXlsUrl: "/test/jeecgDemo/exportXls"
      // },
      // fieldList:superQueryFieldList
    }
  },
  methods: {
    getQueryParams(){
      //高级查询器
      let sqp = {}
      if(this.superQueryParams){
        sqp['superQueryParams']=encodeURI(this.superQueryParams)
        sqp['superQueryMatchType'] = this.superQueryMatchType
      }
      this.isorter = null;
      let param = Object.assign(sqp, this.queryParam, this.isorter ,this.filters);

      param.field = this.getQueryField();
      param.pageNo = this.ipagination.current;
      param.pageSize = this.ipagination.pageSize;
      delete param.birthdayRange; //范围参数不传递后台
      return filterObj(param);
    },
    initDictConfig() {
      // //初始化字典 - 性别
      // initDictOptions('sex').then((res) => {
      //   if (res.success) {
      //     this.sexDictOptions = res.result;
      //   }
      // });
    },
    // onetomany: function () {
    //   this.$refs.jeecgDemoTabsModal.add();
    //   this.$refs.jeecgDemoTabsModal.title = "编辑";
    // },
    //跳转单据页面
    // jump() {
    //   this.$router.push({path: '/jeecg/helloworld'})
    // },
    onBirthdayChange: function (value, dateString) {
      console.log(dateString[0],dateString[1]);
      this.queryParam.birthday_begin=dateString[0];
      this.queryParam.birthday_end=dateString[1];
    },
    //列设置更改事件
    onColSettingsChange (checkedValues) {
      var key = this.$route.name+":colsettings";
      Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
      this.settingColumns = checkedValues;
      const cols = this.defColumns.filter(item => {
        if(item.key =='rowIndex'|| item.dataIndex=='action'){
          return true
        }
        if (this.settingColumns.includes(item.dataIndex)) {
          return true
        }
        return false
      })
      this.columns =  cols;
    },
    initColumns(){
      //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
      //this.defColumns = colAuthFilter(this.defColumns,'testdemo:');

      var key = this.$route.name+":colsettings";
      let colSettings= Vue.ls.get(key);
      if(colSettings==null||colSettings==undefined){
        let allSettingColumns = [];
        this.defColumns.forEach(function (item,i,array ) {
          allSettingColumns.push(item.dataIndex);
        })
        this.settingColumns = allSettingColumns;
        this.columns = this.defColumns;
      }else{
        this.settingColumns = colSettings;
        const cols = this.defColumns.filter(item => {
          if(item.key =='rowIndex'|| item.dataIndex=='action'){
            return true;
          }
          if (colSettings.includes(item.dataIndex)) {
            return true;
          }
          return false;
        })
        this.columns =  cols;
      }
    },


    //demo

    handleCheak(record){
      this.$refs.checkModal.edit(record);
      this.$refs.checkModal.title = "查看";
      this.$refs.checkModal.confirmLoading = false;
    },

    getDemoData(){
      // this.dataSource = [
      //   {
      //     healthCertId,
      //     employeeId,
      //     certNum,
      //     issuingDate,
      //     issuingAuthority,
      //     apartment,
      //     validity,
      //     uploadFileName,
      //     uploadUserId,
      //     uploadDate
      //
      //   }
      // ]
      if(this.dataSource.length != 0){
        for (const dataSourceKey in this.dataSource) {
          dataSourceKey.uploadFileName = dataSourceKey.uploadFileName.split("?");
        }

      }

    },

    getQueryField() {
      //TODO 字段权限控制
      var str = "";
      this.columns.forEach(function (value) {
        str += "," + value.dataIndex;
      });
      return str;
    },





  },
  created() {
    this.initColumns();
    this.getDemoData();
  },
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>