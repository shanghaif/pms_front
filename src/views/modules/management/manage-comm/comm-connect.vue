<template>
  <div class="comm-connect" id="commConnect">
    <el-form :inline="true" :model="secrchDataForm" @keyup.enter.native="getCommconnectList()" class="device-form">
      <el-form-item class="device-form-item" >
        <el-cascader
          :options="organizationOptions"
          v-model="secrchDataForm.orgSelectedOptions"
          change-on-select
          placeholder="组织机构"
        ></el-cascader>
      </el-form-item>
      <!-- <el-form-item class="device-form-item" >
        <el-select 
          v-model="secrchDataForm.channelSelectedOptions" 
          clearable filterable placeholder="选择通道名称" 
          @click.native="getChannel()" 
          :loading="channelLoading">
          <el-option
            v-for="item in channelOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item> -->
      <el-form-item class="device-form-item" >
        <el-select 
          v-model="secrchDataForm.connectTypeSelectedOptions" 
          filterable clearable placeholder="选择连接类型">
          <el-option
            v-for="item in connectTypeOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <!-- <el-form-item class="device-form-item" >
        <el-select 
          v-model="secrchDataForm.connectModelSelectedOptions" 
          clearable filterable placeholder="选择通讯模式">
          <el-option
            v-for="item in connectModelOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item> -->
      <el-form-item class="device-form-item" >
        <el-select
          @click.native='getProtocolTypeOptions()'
          v-model="secrchDataForm.protocolTypeSelectedOptions" 
          clearable filterable placeholder="选择协议名称">
          <el-option
            v-for="item in protocolTypeOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item class="device-form-item" >
        <el-input 
          v-model="secrchDataForm.channelName"
          placeholder="通道名称"
          clearable></el-input>
      </el-form-item>
      <el-form-item class="device-form-item">
        <el-button type="success" @click="getCommconnectList('params')">查询</el-button>
        <el-button v-if="isAuth(OPTAUTH_ADD)" type="primary" @click="addCommconnect()">新增</el-button>
        <el-button v-if="isAuth(OPTAUTH_DELETE)" type="danger" @click="deleteCommconnect()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <div class="connect-list">
      <el-table
        :data="connectDataList"
        border
        :row-style="{height:'36px'}"
        :max-height="connectTableHeight"
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>
        <el-table-column
          type="index"
          header-align="center"
          align="center"
          width="80"
          label="序号">
        </el-table-column>
        <el-table-column
          prop="orgName"
          header-align="center"
          align="center"
          min-width="150"
          label="组织机构名称"
          :show-overflow-tooltip='true'>
        </el-table-column>
        <el-table-column
          prop="channelName"
          header-align="center"
          align="center"
          min-width="120"
          label="通道名称"
          :show-overflow-tooltip='true'>
        </el-table-column>
        <el-table-column
          prop="channelCode"
          header-align="center"
          align="center"
          min-width="120"
          label="通道编号">
        </el-table-column>        
        <!-- <el-table-column
          prop="channelAddress"
          header-align="center"
          align="center"
          min-width="150"
          label="通道物理地址">
        </el-table-column> -->
        <el-table-column
          prop="connectFlag"
          header-align="center"
          align="center"
          min-width="120"
          label="连接标识">
        </el-table-column>
        <el-table-column
          prop="connectType"
          :formatter="changeType"
          header-align="center"
          align="center"
          min-width="120"
          label="连接类型">
        </el-table-column>
        <!-- <el-table-column
          prop="connectModel"
          header-align="center"
          align="center"
          min-width="120"
          label="通讯模式">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.connectModel === 1" size="small" type="danger">主</el-tag>
            <el-tag v-else size="small">从</el-tag>
          </template>
        </el-table-column> -->
        <el-table-column
          prop="protocolName"
          header-align="center"
          align="center"
          min-width="150"
          label="协议名称"
          :show-overflow-tooltip='true'>
        </el-table-column>
        <el-table-column
          prop="localConnectUrl"
          header-align="center"
          align="center"
          min-width="120"
          label="本地IP">
        </el-table-column>
        <el-table-column
          prop="localPort"
          header-align="center"
          align="center"
          min-width="120"
          label="本地端口">
        </el-table-column>
        <!-- <el-table-column
          prop="remoteConnectUrl"
          header-align="center"
          align="center"
          min-width="120"
          label="对侧IP">
        </el-table-column>
        <el-table-column
          prop="remotePort"
          header-align="center"
          align="center"
          min-width="120"
          label="对侧端口">
        </el-table-column> -->
        <el-table-column
          prop="pollingInteval"
          header-align="center"
          align="center"
          min-width="180"
          label="轮询间隔时间（毫秒）">
        </el-table-column>
        <el-table-column
          prop="scadaNo"
          header-align="center"
          align="center"
          min-width="80"
          label="前置标识">
        </el-table-column>
        <el-table-column
          fixed="right"
          header-align="center"
          align="center"
          width="150"
          label="操作">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="dark" content="修改" placement="top">
              <el-button v-if="isAuth(OPTAUTH_UPDATE)" type="text" size="mini" @click="addCommconnect(scope.row)"><i class="el-icon-edit-outline istyle"></i></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="删除" placement="top">
              <el-button v-if="isAuth(OPTAUTH_DELETE)" type="text" size="mini" @click="deleteCommconnect(scope.row.channelId)"><i class="el-icon-delete istyle"></i></el-button>
            </el-tooltip>            
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
        :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        :total="totalPage"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </div>    
    <!-- 弹窗, 新增 / 修改 -->
    <el-dialog
      :title="dialogTitle == undefined ? '新  增' : '修  改'"
      :close-on-click-modal="false"
      :visible.sync="addOrUpdateVisible"
      center="center"
      width="27.66%"
      class="rdialog">
      <el-form :model="dataFormSet" ref="dataFormSet" @keyup.enter.native="dataFormSubmit()" class="rel-form" :rules="dataFormSetRule">
        <el-form-item class="rel-form-item" label="组织机构" prop="setOrganizationOptions">
          <el-cascader
            :options="organizationOptions"
            change-on-select
            v-model="dataFormSet.setOrganizationOptions"
          ></el-cascader>
        </el-form-item>
        <!-- <el-form-item class="set-device-form-item" label="通道标识" prop="channelId">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.channelId"
            clearable>
          </el-input>
        </el-form-item> -->
        <el-form-item class="rel-form-item" prop="channelName" label="通道名称">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.channelName"
            clearable>
          </el-input>
        </el-form-item>
        <el-form-item class="rel-form-item" label="通道编号" prop="channelCode">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.channelCode"
            clearable>
          </el-input>
        </el-form-item>
        <el-form-item class="rel-form-item" label="连接标识" prop="connectFlag">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.connectFlag"
            clearable>
          </el-input>
        </el-form-item>
        <!-- <el-form-item class="set-device-form-item" label="通道物理地址" prop="channelAddress">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.channelAddress"
            clearable>
          </el-input>
        </el-form-item> -->
        <el-form-item class="rel-form-item" label="连接类型" prop="connectType">
          <el-select v-model="dataFormSet.connectType">
            <el-option
              v-for="item in connectTypeOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <!-- <el-form-item class="set-device-form-item" label="通讯模式" prop="connectModel">
          <el-select v-model="dataFormSet.connectModel" @change='changeConnectModel'>
            <el-option
              v-for="item in connectModelOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item> -->
        <el-form-item class="rel-form-item" label="协议名称" prop= "protocolId">
          <el-select v-model="dataFormSet.protocolId" >
            <el-option
              v-for="item in protocolTypeOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>         
        </el-form-item>
        <el-form-item class="rel-form-item" label="本地IP" prop="localConnectUrl">
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.localConnectUrl"
            clearable>
          </el-input>
        </el-form-item>
        <el-form-item class="rel-form-item" label="本地端口" prop="localPort"> 
          <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.localPort"
            clearable
            type="number"
            min="1">
          </el-input>
        </el-form-item>       
        <el-form-item class="rel-form-item" label="轮询间隔时间（毫秒）">
          <el-input-number v-model="dataFormSet.pollingInteval" :step="100" :min="1" controls-position="right"></el-input-number>
        </el-form-item>
        <el-form-item class="rel-form-item" label="前置标识" prop="frontLogo">
          <!-- <el-input
            placeholder="请输入内容"
            v-model="dataFormSet.frontLogo"
            clearable
            type="number"
            min="1">
          </el-input> -->
          <el-select v-model="dataFormSet.frontLogo" @click.native="getScadano()">
            <el-option
              v-for="item in frontLogoOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select> 
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addOrUpdateVisible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit(dataFormSet)">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import { isIpAddress } from '@/utils/validate'
export default {
  data () {
    // 通道名称校验
    var validateChannelName = (rule, value, callback) => {
      // console.log(value)
      if (value && this.dataFormSet.channelName != this.saveRowDataChannelName && this.dataFormSet.channelName != '') {
        this.$http({
          url: this.$http.adornUrl(`/base/tchannel/checkChannelName?channelName=${value}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0 && data.data.status === false) {
            callback(new Error(data.data.msg))
          }else {
            callback()
          }
        })
      } else {
        callback()
      }
    }
    // 连接标识校验
    var validateConnectFlag = (rule, value, callback) => {
      // console.log(value)
      if (value && this.dataFormSet.connectFlag != this.saveRowDataConnectFlag && this.dataFormSet.connectFlag != '') {
        this.$http({
          url: this.$http.adornUrl(`/base/tchannel/checkConnectFlag?connectFlag=${value}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0 && data.data.status === false) {
            callback(new Error(data.data.msg))
          }else {
            callback()
          }
        })
      } else {
        callback()
      }
    }
    // 通道编号校验
    var validateChannelCode = (rule, value, callback) => {
      // console.log(value)
      if (value && this.dataFormSet.channelCode != this.saveRowDataChannelCode && this.dataFormSet.channelCode != '') {
        this.$http({
          url: this.$http.adornUrl(`/base/tchannel/checkChannelCode?channelCode=${value}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0 && data.data.status === false) {
            callback(new Error(data.data.msg))
          }else {
            callback()
          }
        })
      } else {
        callback()
      }
    }
    var validateIpAddress = (rule, value, callback) => {
      if (!isIpAddress(value)) {
        callback(new Error('IP地址不正确'))
      } else {
        callback()
      }
    }
    var validatePort = (rule, value, callback) => {
      if (value > 65535 || value < 0) {
        callback(new Error('地址超出1~65535范围'))
      } else {
        callback()
      }
    }
    return {
      // 增加修改页面协议类型下拉框
      setProtocolTypeOptions: [{
          value: '0',
          label: '104主站协议'
        },{
          value: '1',
          label: '104从站协议'
        },{
          value: '2',
          label: 'Redis主站协议'
        },{
          value: '3',
          label: 'Redis从站协议'
        }],
      // 表格高度
      connectTableHeight: '',
      // 协议类型下拉框  0:104主、1:104从、2:redis主、3:redis从
      protocolTypeOptions: [],
      // 通讯模式下拉框  （1：主，2：从）
      connectModelOptions: [{
          value: '1',
          label: '主'
        },{
          value: '2',
          label: '从'
      }],
      // 通讯类型下拉选框   1：TCP-server，2:TCP-clent，3：RS485-Tcp, 4: UDP
      connectTypeOptions: [{
          value: '0',
          label: 'redis-client'
        },{
          value: '1',
          label: 'TCP-server'
        },{
          value: '2',
          label: 'TCP-client'
        },{
          value: '3',
          label: 'RS485-TCP'
        },{
          value: '4',
          label: 'UDP'
      }],
      // 增加、修改页面表单
      dataFormSet: {
        setOrganizationOptions: [],
        channelName: null,
        channelCode: null,
        connectFlag: null,
        connectType: null,
        protocolId: null,
        localConnectUrl: null,
        localPort: null,
        pollingInteval: null,
        frontLogo: null
      },
      dataFormSetRule: {
        localConnectUrl: [
          { required: true, message: '本地IP不能为空', trigger: 'blur' },
          { validator: validateIpAddress, trigger: 'blur' }
        ],
        remoteConnectUrl: [
          { required: true, message: '对侧IP不能为空', trigger: 'blur' },
          { validator: validateIpAddress, trigger: 'blur' }
        ],
        setOrganizationOptions: [{ required: true, message: '组织机构不能为空', trigger: 'blur' }],
        channelName: [
          { required: true, message: '通道名称不能为空', trigger: 'blur' },
          { validator: validateChannelName, trigger: 'blur'}
        ],
        channelCode: [
          { required: true, message: '通道编号不能为空', trigger: 'blur' },
          { validator: validateChannelCode, trigger: 'blur' }
        ],
        connectFlag: [
          { required: true, message: '连接标识不能为空', trigger: 'blur' },
          { validator: validateConnectFlag, trigger: 'blur' }
        ],
        channelAddress: [{ required: true, message: '通道物理地址不能为空', trigger: 'blur' }],
        localPort: [
          { required: true, message: '本地端口不能为空', trigger: 'blur' },
          { validator: validatePort, trigger: 'blur' }
        ],
        remotePort: [
          { required: true, message: '对侧端口不能为空', trigger: 'blur' },
          { validator: validatePort, trigger: 'blur' }
        ],
        connectType: [{ required: true, message: '通讯类型不能为空', trigger: 'blur' }],
        protocolId: [{ required: true, message: '协议名称不能为空', trigger: 'blur' }],
        frontLogo: [{ required: true, message: '前置标识不能为空', trigger: 'blur' }]
      },
      // 区分是新增还是修改页面
      dialogTitle: '',
      // 搜索表单 分别为已选组织机构、已选通道标识
      secrchDataForm: {
        orgSelectedOptions: [],
        connectTypeSelectedOptions: null,
        protocolTypeSelectedOptions: null,
        channelName: null
      },
      // 组织机构列表
      organizationOptions: [],
      // 通道标识列表
      channelOptions: [],
      // 表格数据
      connectDataList: [],
      // 当前页
      pageIndex: 1,
      // 分页大小
      pageSize: 10,
      // 总记录数
      totalPage: 50,
      // 多选数组
      dataListSelections: [],
      addOrUpdateVisible: false,
      dataListLoading: false,
      channelLoading: false,
      connectTypeLoading: false,
      // 点击修改某行时 保存信息
      saveRowDataChannelName: null,
      saveRowDataConnectFlag: null,
      saveRowDataChannelCode: null,
      frontLogoOptions: []  // 前置标识下拉框数据
    }
  },
  created () {
    this.connectTableHeight = window.innerHeight-320    
  },
  watch: {
    /* 'secrchDataForm.connectModelSelectedOptions': function (val) {
      this.secrchDataForm.protocolTypeSelectedOptions = null
    }, */
    /* 'dataFormSet.connectModel': function (val) {
      this.dataFormSet.protocolType = null
    } */
  },
  mounted () {
    this.getOrgLists()
    this.getCommconnectList()
    // document.getElementById('commConnect').style.height = (window.innerHeight-120)+'px'
    window.onresize = function () {
      if(document.getElementById('commConnect')){
        // document.getElementById('commConnect').style.height = (window.innerHeight - 120) + 'px'
        this.connectTableHeight = window.innerHeight-320
      }
    }    
  },
  methods: {
    // 增加修改页面点击通讯模式，清除协议类型选项
    changeConnectModel (val) {
      this.dataFormSet.protocolId = null
    },
    // 协议类型下拉框
    getProtocolTypeOptions () {
      // console.log(this.addOrUpdateVisible)
      this.$http({
        url: this.$http.adornUrl('/admin/tprotocol/protocolname'),
        method: 'post',
        data: this.$http.adornParams({})
      }).then(({data}) => {
        if (data && data.code === 0) {
          // console.log(data.data)
          this.protocolTypeOptions = data.data
        }
      })      
    },
    // 提交修改配置页面信息
    dataFormSubmit (dataFormSet) {
      // console.log(dataFormSet)
      // console.log(this.saveRowData)
      Promise.all([
        this.checkChannelName(),
        this.checkConnectFlag(),
        this.checkChannelCode()
      ]).then((res)=>{
        // console.log(res)
        // if(res[0].data && data.code === 0 && data.data.status === true)
        if((res && res.length == 3 && res[0] && res[1] && res[2]) || (res && res[0] == 1 && res[1] == 1 && res[2] ==1)) {
          this.$refs['dataFormSet'].validate((valid) => {
            if (valid) {
              this.$confirm(`确定提交?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
              }).then(() => {
                this.$http({
                  url: this.$http.adornUrl(`/base/tchannel/${this.dialogTitle == undefined ? 'save' : 'update'}`),
                  method: 'post',
                  data: this.$http.adornData({
                    'channelId':this.dialogTitle == undefined ? null : this.dialogTitle,
                    'orgId': dataFormSet.setOrganizationOptions[dataFormSet.setOrganizationOptions.length-1],
                    'channelName': dataFormSet.channelName,
                    'channelCode': dataFormSet.channelCode,
                    'connectFlag': dataFormSet.connectFlag,
                    'connectType': dataFormSet.connectType,
                    'protocolId': dataFormSet.protocolId,
                    'localConnectUrl': dataFormSet.localConnectUrl,
                    'localPort': dataFormSet.localPort,
                    'pollingInteval': dataFormSet.pollingInteval,
                    'scadaNo': dataFormSet.frontLogo
                  })
                }).then(({data}) => {
                  if(data && data.code === 0) {
                    this.addOrUpdateVisible = false
                        // if(this.dialogTitle == undefined) {
                        //   this.connectDataList.unshift(data.entity)
                        // } else {
                        //   for(let i=0;i<this.connectDataList.length;i++) {
                        //     if(this.connectDataList[i].channelId == data.entity.channelId) {
                        //       this.connectDataList[i].orgId = data.entity.orgId
                        //       this.connectDataList[i].channelName = data.entity.channelName
                        //       this.connectDataList[i].channelCode = data.entity.channelCode
                        //       this.connectDataList[i].connectFlag = data.entity.connectFlag 
                        //       this.connectDataList[i].connectType = data.entity.connectType
                        //       this.connectDataList[i].protocolName = data.entity.protocolName
                        //       this.connectDataList[i].localConnectUrl = data.entity.localConnectUrl
                        //       this.connectDataList[i].localPort = data.entity.localPort
                        //       this.connectDataList[i].pollingInteval = data.entity.pollingInteval
                        //       this.connectDataList[i].scadaNo = data.entity.scadaNo
                        //     }
                        //   }
                        // }
                        this.$refs['dataFormSet'].resetFields()
                        this.$options.methods.getCommconnectList.bind(this)()
                    this.$message({
                      message: '操作成功',
                      type: 'success',
                      duration: 1500,
                      onClose: () => {
                      }
                    })                
                  } else {
                    this.$message.error(data.msg)
                  }
                })            
              })
            } else {
              console.log('error submit!!')
              return false;
            }
          }) 
        }
        

      })     
    },
    checkChannelName () {
      let that = this 
      let p = new Promise(function(resolve, reject){  
        if(that.saveRowDataChannelName != that.dataFormSet.channelName) {
          that.$http({
            url: that.$http.adornUrl(`/base/tchannel/checkChannelName?channelName=${that.dataFormSet.channelName}`),
            method: 'get',
            params: that.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0 && data.data.status === true) {
              resolve(data)
            } else if(data && data.code === 0 && data.data.status === false){
              that.$message({
                message: '通道名称已存在',
                type: 'success',
                duration: 1500,
                onClose: () => {}
              })
            }
          })
        } else {
          resolve('1')
        }
      })
      return p 
    },
    checkConnectFlag() {
      let that = this 
      let p = new Promise(function(resolve, reject){  
        if(that.saveRowDataConnectFlag != that.dataFormSet.connectFlag) {
          that.$http({
            url: that.$http.adornUrl(`/base/tchannel/checkConnectFlag?connectFlag=${that.dataFormSet.connectFlag}`),
            method: 'get',
            params: that.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0 && data.data.status === true) {
              resolve(data)
            } else if(data && data.code === 0 && data.data.status === false) {
              that.$message({
                message: '通道标识已存在',
                type: 'success',
                duration: 1500,
                onClose: () => {}
              })
            }
          })
        } else {
          resolve('1')
        }
      })
      return p 
    },
    checkChannelCode () {
      let that = this 
      let p = new Promise(function(resolve, reject){  
        if(that.saveRowDataChannelCode != that.dataFormSet.channelCode) {
          that.$http({
            url: that.$http.adornUrl(`/base/tchannel/checkChannelCode?channelCode=${that.dataFormSet.channelCode}`),
            method: 'get',
            params: that.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0 && data.data.status === true) {
              resolve(data)
            } else if(data && data.code === 0 && data.data.status === false) {
              that.$message({
                message: '通道编号已存在',
                type: 'success',
                duration: 1500,
                onClose: () => {}
              })
            }
          })
        } else {
          resolve('1')
        }
      })
      return p 
    },
    // 获取组织机构数据
    getOrgLists () {
      this.$http({
        url: this.$http.adornUrl('/admin/tstation/org'),
        method: 'post',
        data: this.$http.adornParams({})
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.organizationOptions = data.list
          if(this.secrchDataForm.orgSelectedOptions.length == 0) {
            this.secrchDataForm.orgSelectedOptions = JSON.parse(window.sessionStorage.getItem('userInfo')).orgId.split() 
          }      
        }
      })      
    },
    // 获取通道标识
    /* getChannel () {
      this.channelLoading = true           
      this.$http({
        url: this.$http.adornUrl('/admin/tprotocolredismaster/channel'),
        method: 'post',
        data: this.$http.adornParams({
          orgId: this.secrchDataForm.orgSelectedOptions.length == 0 ? JSON.parse(window.sessionStorage.getItem('userInfo')).orgId : this.secrchDataForm.orgSelectedOptions[this.secrchDataForm.orgSelectedOptions.length -1]
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.channelLoading = false
          this.channelOptions = data.list                 
        }
      })
    }, */
    // 获取数据列表
    getCommconnectList (params) {
      if(params){
          this.pageIndex=1
        }
      // console.log(this.secrchDataForm)
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/base/tchannel/list'),
        method: 'post',
        data: this.$http.adornParams({
          'pageNumber': this.pageIndex.toString(),
          'pageSize': this.pageSize.toString(),
          'entity': {         
            'orgId': this.secrchDataForm.orgSelectedOptions.length == 0 ? JSON.parse(window.sessionStorage.getItem('userInfo')).orgId : this.secrchDataForm.orgSelectedOptions[this.secrchDataForm.orgSelectedOptions.length -1],
            'connectType': this.secrchDataForm.connectTypeSelectedOptions, // "通讯连接"
            "protocolId" : this.secrchDataForm.protocolTypeSelectedOptions,// "协议标识",
            "channelName" : this.secrchDataForm.channelName// "通道名称"
          }
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.connectDataList = data.page.list
          this.totalPage = data.page.total
          this.pageIndex = data.page.pageNum
          this.pageSize = data.page.pageSize
        } else {
          this.connectDataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getCommconnectList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getCommconnectList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    getScadano () {
      this.$http({
        url: this.$http.adornUrl('/admin/tprotocol/scadaNo'),
        method: 'post',
        data: this.$http.adornParams({
          'orgId': this.dataFormSet.setOrganizationOptions[this.dataFormSet.setOrganizationOptions.length-1]
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.frontLogoOptions = data.data            
        }
      })
    },
    // 新增 / 修改
    addCommconnect (row) {
      this.addOrUpdateVisible = true
      this.$nextTick(function(){
        this.$refs["dataFormSet"].resetFields()
        this.dataFormSet.pollingInteval = '200'
        // this.$options.methods.getProtocolTypeOptions.bind(this)()
      })
      this.$http({
        url: this.$http.adornUrl('/admin/tprotocol/protocolname'),
        method: 'post',
        data: this.$http.adornParams({})
      }).then(({data}) => {
        if (data && data.code === 0) {
          // console.log(data.data)
          this.protocolTypeOptions = data.data
        }
      }) 
      // 修改则把改行信息传入 增加则没有
      if(row) {        
        this.$http({
          url: this.$http.adornUrl('/admin/tstation/orgName'),
          method: 'post',
          data: this.$http.adornData({
            orgId: row.orgId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataFormSet.setOrganizationOptions = data.list
          }else {
            this.dataFormSet.setOrganizationOptions = []
          }
        })
        this.$http({
          url: this.$http.adornUrl(`/base/tchannel/info?channelId=${row.channelId}`),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.saveRowDataChannelName = data.tChannel.channelName
            this.saveRowDataChannelCode = data.tChannel.channelCode
            this.saveRowDataConnectFlag = data.tChannel.connectFlag
            this.dialogTitle = data.tChannel.channelId
            this.dataFormSetsetOrganizationOptions = data.tChannel.orgId
            this.dataFormSet.channelName = data.tChannel.channelName
            this.dataFormSet.channelCode = data.tChannel.channelCode
            this.dataFormSet.connectFlag = data.tChannel.connectFlag
            this.dataFormSet.connectType = data.tChannel.connectType.toString()
            this.dataFormSet.protocolId = data.tChannel.protocolId
            this.dataFormSet.localConnectUrl = data.tChannel.localConnectUrl
            this.dataFormSet.localPort = data.tChannel.localPort
            this.dataFormSet.pollingInteval = data.tChannel.pollingInteval
            this.dataFormSet.frontLogo = data.tChannel.scadaNo
            this.$http({
              url: this.$http.adornUrl('/admin/tprotocol/scadaNo'),
              method: 'post',
              data: this.$http.adornParams({
                'orgId': this.dataFormSet.setOrganizationOptions[this.dataFormSet.setOrganizationOptions.length-1]
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.frontLogoOptions = data.data            
              }
            })
          }else {
            this.dataFormSet = []
          }
        })
      }else {
        this.dialogTitle = row
        this.$http({
          url: this.$http.adornUrl('/base/tchannel/getChannelCode'),
          method: 'get',
          params: this.$http.adornParams({})
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataFormSet.channelCode = data.data
          }else {
          }
        })
      }
    },
    // 删除
    deleteCommconnect (id) {
      var channelId = id ? [id] : this.dataListSelections.map(item => {
        return item.channelId
      })
      this.$confirm('确定删除操作?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/base/tchannel/delete'),
          method: 'post',
          data: this.$http.adornData(channelId, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
                this.getCommconnectList()
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {})
    },
    // 通讯类型（0:无，1：TCP-server，2:TCP-clent，3：RS485）
    changeType (cellValue) {
      if(cellValue.connectType === 0){
        return 'redis-client'
      }else if(cellValue.connectType === 1){
        return 'TCP-server'
      }else if(cellValue.connectType === 2){
        return 'TCP-client'
      }else if(cellValue.connectType === 3){
        return 'RS485-TCP'
      }else if(cellValue.connectType === 4){
        return 'UDP'
      }
    },
    // 通讯模式（1：主，2：从）
    changeModel (cellValue) {
      if(cellValue.connectModel === '1'){
        return '主'
      }else if(cellValue.connectModel === '2'){
        return '从'
      }else {
        return '-'
      }
    },
    // 协议类型  0:104主、1:104从、2:redis主、3:redis从
    protocolType (cellValue) {
      if(cellValue.protocolId === '0') {
        return '104主站协议'
      }else if(cellValue.protocolId === '1') {
        return '104从站协议'
      }else if(cellValue.protocolId === '2') {
        return 'Redis主站协议'
      }else if(cellValue.protocolId === '3') {
        return 'Redis从站协议'
      }
    }
  }
}
</script>
<style scoped>
/* 搜索条件栏样式开始 */
.device-form {
 /*  background-color: #eef4fa; */
  border-radius: 5px 5px 0 0 ;
  /* margin-bottom: 10px; */
  height: 80px;
  width: 100%;
  padding: 0 10px 0 10px;
  display: flex;
  align-items: center;
  text-align: justify
}
.device-form-item {
  margin-bottom: 0;
  /* flex: 1 */
}
.set-device-form-item {
  font-weight: 700;
  font-size: 15px
}
/* 搜索条件栏样式结束 */

/* 表格样式开始 */
.connect-list {
  /* width: 100%;
  height: 90%;
  min-height: 90%;
  margin: 0 auto;
  font-size: 14px;
  overflow-y: auto */
}

/* 表格样式结束 */

/* 增加修改配置页面样式开始 */
/* .comm-connect >>> .el-dialog {
  height: 70%;
}
.comm-connect >>> .el-dialog .el-dialog__body {
  height: 80%;
} */
.set-device-form {
  background-color: #eef4fa;
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  width: 100%;
  padding: 20px 0 0 1%
  /* height: 100%;
  overflow-y: auto */
}
.set-device-form-item >>> .el-form-item__content {
  width: 50%;
  display: flex;
  flex-direction: row;
}
/* .set-device-form-item a {
  flex: 1;
  min-width: 150px;
  text-decoration: none
} */
.set-device-form-item >>> .el-select, .set-device-form-item >>> .el-cascader, .set-device-form-item >>> .el-input{
  flex: 3
}
.set-device-form-item >>> .el-form-item__label {
  min-width: 180px
}

.set-protocolType-form {
  display: flex;
  flex-direction: column;
  padding: 20px 0 0 0;
  background-color: #DCDFE6
}
.set-protocolType-form-item >>> .el-form-item__content{
  width: 50%;
  display: flex;
  flex-direction: row;
}
.set-protocolType-form-item >>> .el-form-item__label {
  min-width: 150px
}
.set-protocolType-form-item >>> .el-select,  .set-protocolType-form-item >>> .el-input{
  flex: 3;
  margin-bottom: 1%
}
.set-protocolType-form-item >>> label + .el-form-item__content {
  padding-top: 1%
}
.set-protocolType-form-item >>> .el-radio {
  flex: 2;
  justify-content: center;
  display: flex;
  align-items: center;
}

/* .comm-connect >>> table thead tr th:nth-last-child(1) {
  border-right: none
} */

/* 增加修改配置页面样式结束 */

/* .istyle {
  font-size: 18px;
  font-weight: 500;
  color: #3397ff
}
.comm-connect >>> .el-pager li.active {
  background-color: #33a7fe;
  color: #fff;
  font-size: 16px;
  border-radius: 100%
}
.comm-connect >>> .el-pager li {
  min-width: 30px;
  height: 25px;
  line-height: 25px
} */
/* .comm-connect >>> .el-table td, .comm-connect >>> .el-table th {
  padding: 0
} */
.comm-connect >>> .el-dialog--center {
  margin-top: 5vh !important
}
</style>
<style lang="scss" scoped>
@import "../../../../assets/scss/_dialog.scss"
</style>