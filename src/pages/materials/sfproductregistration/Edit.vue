﻿<template>
<div>
  <el-row class="yl-order-header">
    <el-col :span="2">
      <img src="../../../common/image/zgztlogo.png" alt="logo" class="yl-order-logo">
      </el-col>
    <el-col :span="18" > 
     <div class="yl-order-title"><span class="yl-order-small-title">{{title}}&nbsp;&nbsp;&nbsp;</span>半成品登记</div>
    </el-col>
      <el-col :span="4">
      <div class="yl-order-code"><span style="color: red;">NO.</span>{{formModel.order.orderCode}}</div>
      </el-col>
 </el-row>
  <el-form  :model="formModel.order"  :rules="rules" ref="formModel" label-width="100px" >
    <el-row>
        <el-col :span="8"> 
            <el-form-item label="记账单位" prop="orgId">
              <yl-organizeTree  
                      v-model="formModel.order.orgId"  
                      :placeholder="'请选择单位'"   
                      :isexpand="true"
                      @getCurrentNode="_getCurrentNodeOrg"  
                      style="width:80%">
                      </yl-organizeTree>
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="账期" prop="orderDate">
                <el-date-picker
                        v-model="formModel.order.orderDate"
                        @change="_dateSel"
                        type="month"
                        :editable="false"
                        placeholder="请选择"
                        style="width:80%">
                </el-date-picker>
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="生产班组" prop="productTeam">
                <yl-comdatadictionary :code="'ProductTeam'" v-model="formModel.order.productTeam" :readonly="true" :placeholder="'请选择'"  style="width:80%">
                 </yl-comdatadictionary>
            </el-form-item>
       </el-col>
    </el-row>
         

     <el-row class="yl-row-TableEditor">
      <el-col :span="24"> 
        <yl-tableedit ref="tableedit"
            :tableConfig="tableEditConfig"
            :tableData="formModel.items"
            :sumRowConf="sumRowConf"
            :mapConfig="tableEditMapConfig"
            @delItem="_delItem"
        ></yl-tableedit>
       </el-col>
       
     </el-row>

   <el-row>
        <el-form-item label="备注" prop="remark">
            <el-input v-model="formModel.order.remark" type="textarea" style="width:90%" ></el-input>
        </el-form-item>
   </el-row>
    <el-row>
        <el-col :span="8"> 
            <el-form-item label="签字人一：" prop="personOne">
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="签字人二：" prop="personTwo">
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="制单人：" prop="maker">
                <span>{{formModel.order.maker}}</span>
            </el-form-item>
        </el-col>
    </el-row>
   <el-form-item style="text-align:right;" >
      <el-button type="primary" @click="_onSelect"><i class="icon-plus"></i>选材</el-button>
      <el-button type="primary" @click="_onSubmit" :loading="loading"><i class="icon-floppy-disk"></i>保存</el-button>
  </el-form-item>
  
</el-form>


   <el-dialog ref="selectDialog2" title="选择"  
                 v-model="addFormVisible" size="small"  
                 top="8%" 
                 :modal="false"
                 :lock-scroll="true"
                 :close-on-press-escape="false"
                 :modal-append-to-body="false" 
                 :close-on-click-modal="false"  >
                <yl-selectfromsproduct    
                    :isSelectClose="true"
                    @close="_close"
                    v-if="addFormVisible">
                </yl-selectfromsproduct>
  </el-dialog>


</div>
</template>
<script type="text/babel">
import selectFromSProduct from 'ocomponents/selectmodule/selectFromSProduct'
import config from 'static/js/config';
import {
            requestCreateOrUpdateSFProductRegistrationOrder,
            requestGetSFProductRegistrationOrderForEdit,
            requestDeleteSFProductRegistrationOrderItem,
        }   from 'api/materials/sfproductregistrationorder';
import organizeTree from 'ocomponents/organize/organizeTree';
import comdatadictionary from 'ocomponents/comdatadictionary/ComDataDictionary'
export default {
    data(){
        return{
            formModel:{
                    order:{
                        orderDate:new Date(),  // orderDate 需要初始化 
                        orgId:this.getUserInfo().user.manageOrgId,
                             orderId:'',
                             orgName:this.getUserInfo().user.manageOrgName,
                             orderCode:'',
                             dataType:'',
                             dataId:'',
                             productTeam:'',
                             isAudit:'',
                             auditor:'',
                             auditDate:'',
                             isSate:0,
                             maker:'',
                             makerDate:'',
                             personOne:'',
                             personTwo:'',
                             personThree:'',
                             personFour:'',
                             personFive:'',
                             remark:'',
                             extendColumn1:'',
                             extendColumn2:'',
                             extendColumn3:'',
                        },
                        items:[],
                        config:{isGenerateCode:true},
            },
            rules: {
                   
                            orgId: [
                              { required: true, message: '记账单位不能为空！', trigger: 'change' }
                            ],
                           
                            orderDate: [
                              { type:"date", required: true, message: '账期不能为空！', trigger: 'change' }
                            ],
                          
                            productTeam: [
                              { required: true, message: '生产班组不能为空！', trigger: 'change' }
                            ],
                        
            },
            sumRowConf:{
                            disable:false, //是否显示
                            sumText:'合计', //合计文本
                            items:{
                              //合计项配置
                              //  planQuantity:{text:'数量合计',sumval:0.00,capitalCol:false},
                              //  lossQuantity:{text:'损耗量合计',sumval:0.00,capitalCol:false},
                              //  responseSum:{text:'金额合计',sumval:0.00,capitalCol:true}
                            }
                        },
            loading:false,
            addFormVisible:false,
            orderDateInital:'',
            title:config.Order.title,
        }       
    },
    props:{
        selectRow:{
            type:Object,
            default:{},
        },
        isVisible:{
             type:Boolean,
             default:false,
        }
    },
    computed: { 
        tableEditConfig:{
                get () {
                    return {
                        table: {
                            attr: {
                                highlightCurrent:true,
                                style:{width:'100%',height:'300px'},
                            }
                        },
                        columns: [
                            {attr: { type: 'index',label: '序号', width:65,align: 'center' }},
                             { attr: { prop: 'id', label: '操作', width: 65, scopedSlot: 'id', } },
                            //列配合
                                // {attr: { prop: 'itemId', label: '', width:100,  } },   
                                // {attr: { prop: 'orgId', label: '', width:100,  } },   
                                // {attr: { prop: 'orderId', label: '', width:100,  } },   
                                // {attr: { prop: 'sInfoNM', label: '', width:100,  } },   
                                // {attr: { prop: 'sInfoCode', label: '材料编码', width:100,  } },   
                                {attr: { prop: 'sInfoName', label: '材料名称', width:150,  } },   
                                {attr: { prop: 'sInfoModel', label: '规格型号', width:150,  } },   
                                {attr: { prop: 'sInfoUnit', label: '单位', width:100,  } },   
                                // {attr: { prop: 'barCode', label: '条码', width:100,  } },   
                                {attr: { prop: 'quantity', label: '数量', width:120, scopedSlot:"quantity",  } }, 
                                {attr: { prop: 'detectedInfo', label: '检测信息', width:150,  scopedSlot:"detectedInfo",} },  
                                {attr: { prop: 'productDate', label: '生产日期', width:150, scopedSlot:"productDate", } },  
                                {attr: { prop: 'remark', label: '备注',  scopedSlot:"remark", } },   
                                // {attr: { prop: 'dataType', label: '', width:100,  } },   
                                // {attr: { prop: 'dataId', label: '', width:100,  } },   
                                // {attr: { prop: 'extendColumn1', label: '', width:100,  } },   
                                // {attr: { prop: 'extendColumn2', label: '', width:100,  } },   
                                // {attr: { prop: 'extendColumn3', label: '', width:100,  } },   
                                    
                            //注意：最后一列如果不设置列宽将会自动宽度补全
                        ],
                        
                        
                        //编辑列具名配置
                        solSlotConf:[ {  type:'input',   // 类型  目前 支持 input switch datepick
                                         name:'quantity',  //字段名称
                                        elmentConfig:{  //控件配置   参照elementui配置参数
                                                maxlength:1000000, 
                                                minlength:0,
                                                size:"small",
                                                disabled:false,
                                                type:"text",
                                                placeholder:"",
                                                icon:"",
                                                readonly:false, }, 
                                         eventconf:{     //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.quantity)){
                                                        
                                                    }else{
                                                         row.quantity=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                                         {  type:'input',   // 类型  目前 支持 input switch datepick
                                         name:'productDate',  //字段名称
                                        elmentConfig:{  //控件配置   参照elementui配置参数
                                                maxlength:1000000, 
                                                minlength:0,
                                                size:"small",
                                                disabled:false,
                                                type:"text",
                                                placeholder:"",
                                                icon:"",
                                                readonly:false, }, 
                                         eventconf:{     //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    
                                                },
                                             }
                                         },
                                         {  type:'input',   // 类型  目前 支持 input switch datepick
                                         name:'remark',  //字段名称
                                        elmentConfig:{  //控件配置   参照elementui配置参数
                                                maxlength:1000000, 
                                                minlength:0,
                                                size:"small",
                                                disabled:false,
                                                type:"text",
                                                placeholder:"",
                                                icon:"",
                                                readonly:false, }, 
                                         eventconf:{     //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.quantity)){
                                                        
                                                    }else{
                                                         row.quantity=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                                         {  type:'input',   // 类型  目前 支持 input switch datepick
                                         name:'detectedInfo',  //字段名称
                                        elmentConfig:{  //控件配置   参照elementui配置参数
                                                maxlength:1000000, 
                                                minlength:0,
                                                size:"small",
                                                disabled:false,
                                                type:"text",
                                                placeholder:"",
                                                icon:"",
                                                readonly:false, }, 
                                         eventconf:{     //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.quantity)){
                                                        
                                                    }else{
                                                         row.quantity=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                                    
                        ],
                        
                    }
                }
            },
        tableEditMapConfig:{
             get () {
                 return {
                     mapconf: [   
                                     //明细编辑列和选材控件列映射配置
                                { prop: 'id', isplace: false, defaultval:'0',selprop: '' },    //主键列
                                { prop: 'dataId', isplace: true, defaultval:'',selprop: 'id' }, //来源ID
                                { prop: 'dataType', isplace: false, defaultval:'0',selprop: '' },  //来源类型  // 0 常用材料库 1 进料登记选材 2 从总需用计划选材  3 进料登记（30进料登记PDA选材 31发料登记选材 32内部调拨登记选材 33外部调拨登记选材 ）4从磅单选材 5 合同选材 6从库存选材     
                                { prop: 'guid', isplace: false, defaultval:this.newGuid(),selprop: '' },  //临时ID唯一键
                                { prop: 'itemId', isplace: false,defaultval:'', selprop:'itemId',  },   
                                { prop: 'orgId', isplace: false,defaultval:'', selprop:'orgId',  },   
                                { prop: 'orderId', isplace: false,defaultval:'', selprop:'orderId',  },   
                                { prop: 'sInfoNM', isplace: true,defaultval:'', selprop:'id',  },   
                                { prop: 'sInfoCode', isplace: true,defaultval:'', selprop:'sInfoCode',  },   
                                { prop: 'sInfoName', isplace: true,defaultval:'', selprop:'sInfoName',  },   
                                { prop: 'sInfoModel', isplace: true,defaultval:'', selprop:'sInfoModel',  },   
                                { prop: 'sInfoUnit', isplace: true,defaultval:'', selprop:'sInfoUnit',  },   
                                { prop: 'barCode', isplace: false,defaultval:'', selprop:'barCode',  },   
                                { prop: 'quantity', isplace: false,defaultval:'', selprop:'quantity',  },   
                                { prop: 'productDate', isplace: false,defaultval:'', selprop:'productDate',  },   
                                { prop: 'detectedInfo', isplace: false,defaultval:'', selprop:'detectedInfo',  },   
                                { prop: 'remark', isplace: true,defaultval:'', selprop:'remark',  },   
                                { prop: 'dataType', isplace: true,defaultval:'', selprop:'dataType',  },   
                                { prop: 'dataId', isplace: true,defaultval:'', selprop:'id',  },   
                                { prop: 'extendColumn1', isplace: true,defaultval:'', selprop:'extendColumn1',  },   
                                { prop: 'extendColumn2', isplace: true,defaultval:'', selprop:'extendColumn2',  },   
                                { prop: 'extendColumn3', isplace: true,defaultval:'', selprop:'extendColumn3',  },   
                                    
                               
                               
                            ] 
                 }
             }
         },
    }, 
    methods:{
         _sumCount(colName){
             //计算列的合计值  合计方法计算逻辑
          //  let planQuantityTotal=0.00,lossQuantityTotal=0.00,responseSumTotal=0.00;
          //  this.formModel.items.map((item)=>{
                //凡是手动输入后的input都会转化为str  因此计算时要转化一下。
              //  planQuantityTotal=planQuantityTotal+parseFloat(item.planQuantity);
              //  lossQuantityTotal=lossQuantityTotal+parseFloat(item.lossQuantity);
              //  responseSumTotal=responseSumTotal+item.responseSum;
        //    })
           // this.sumRowConf.items.planQuantity.sumval=planQuantityTotal.toFixed(2);
           // this.sumRowConf.items.lossQuantity.sumval=lossQuantityTotal.toFixed(2);
          //  this.sumRowConf.items.responseSum.sumval=responseSumTotal.toFixed(2);
         },
         _formInitial(){
            //初始化表单
            requestGetSFProductRegistrationOrderForEdit(this.selectRow.id).then(data=>{
                if(data.success){
                     Object.assign(this.formModel,data.result); 
                     this.orderDateInital=this.formModel.order.orderDate;
                     this.formModel.order.orderDate=new Date(this.formModel.order.orderDate);
                     this._sumCount();
                }else{
                     this.$message.error('获取编辑表信息失败！');
                }
            });
          },
         _delItem(row){
                this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning',
                modal:false
                }).then(() => {
                        if(row.id!=="0"){
                            //后台删除
                            let obj={};
                            obj.id=row.id;
                            requestDeleteSFProductRegistrationOrderItem(obj).then(data=>{
                                if(data.success){
                                     let index=this.formModel.items.indexOf(row);
                                     this.formModel.items.splice(index,1);
                                     // this._sumCount();
                                    this.$notify({
                                            title: '成功',
                                            message: '删除数据成功！',
                                            type: 'success'
                                            });
                            }else {
                                this.$notify.error({
                                    title: '错误',
                                      message: '删除数据失败！'+data.error.message,
                                    });
                             }
                            })
                        }else{
                         //前台删除
                            let index=this.formModel.items.indexOf(row);
                            this.formModel.items.splice(index,1);
                            //this._sumCount();
                            this.$notify({
                                            title: '成功',
                                            message: '删除数据成功！',
                                            type: 'success'
                                            });
                        }
                       
                }).catch(()=>{});
            },
         _onSubmit() {
                let _this=this;
                this.formModel.order.orderDate=new Date(this.formModel.order.orderDate);
                this.$refs.formModel.validate((valid) => {
                    if (valid) {
                        this.formModel.items=this.$refs.tableedit.getEditData();
                        if(this.formModel.items.length==0){
                              this.$message({ message: `请选择材料明细！`, type: 'warning'});
                              return;
                        }
                        this.formModel.order.orderDate=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
                        this.formModel.order.isAudit=false;
                        if(this.formModel.order.orderId!==""){
                            //修改
                             if(this.orderDateInital==this.formModel.order.orderDate){
                                 this.formModel.config.isGenerateCode=false;
                             }
                        }
                        this.loading=true;
                        requestCreateOrUpdateSFProductRegistrationOrder(this.formModel).then((data)=>{
                            if(data.success){
                                this.$notify({
                                    title: '成功',
                                    message: '保存数据成功！',
                                    type: 'success'
                                    });
                                //关闭面板
                                this._complete();
                            } 
                            else {
                                 this.$message.error('保存数据失败！'+data.error.message);
                            }
                            this.loading=false;
                        }).catch(function(error){
                                _this.loading=false;
                        });
                    }else{
                         this.$message({ message: `请按照条件填写表单！`, type: 'warning'});
                    }
                });
            },
         _resetForm() {
                this.$refs.formModel.resetFields();
          },
         _complete(){
                this.$emit('close');
           },
         _close(){
                //关闭选材框
                this.addFormVisible=false;
                this._sumCount();
          },
         _onSelect(){
             //打开选材框的权限判断
             this.tableEditMapConfig=this.MapConfig;
             this.addFormVisible=true;
          },
        _getCode(val){
             this.generateOrderCode('SP_SFProductRegistrationOrder',this.formModel.order.orgId,val,(val)=>{
                     this.formModel.order.orderCode=val;
             });
         },
          _getCurrentNodeOrg(data){
             //组织机构选择事件
            this.formModel.order.OrgName=data.text;
            let val=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
            this._getCode(val);
          },
        _dateSel(val){
           this._getCode(val);
        }
    },
    mounted(){
        if(this.selectRow.id!=undefined){
             //修改
             this._formInitial();
         } else{
             let val=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
             this._getCode(val);
         }
    },    
    components:{
       'yl-organizeTree':organizeTree,
       'yl-comdatadictionary':comdatadictionary,
       'yl-selectfromsproduct':selectFromSProduct,
    },
    watch:{
        
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
