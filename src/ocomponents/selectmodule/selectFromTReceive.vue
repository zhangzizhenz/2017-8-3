<template>
    <yl-selectcontainer    
        :tableConfig="tableSelectConfig"
        :filter="tableSelectfilter"
        :funconfig="tableSelectfunconfig"
        :isSelectClose="isSelectClose"
        :dataType="80"
        @close="_close"
        >
    </yl-selectcontainer>
</template>

<script type="text/babel">
export default {
    props:{
        isSelectClose:{
            type:Boolean,
            default:false,
        }
    },
    computed: { 
        //表格配置
        tableSelectConfig:{
                get () {
                    return {
                        table: {
                            attr: {
                                data:{},
                                highlightCurrent:false,
                                style:{width:'100%',height:'300px'},
                            }
                        },
                        columns: [
                                {attr: { type: 'selection',width:55, }},  
                                {attr: { prop: 'infoName', label: '名称', width:120,  } },   
                                {attr: { prop: 'infoModel', label: '规格型号', width:120,  } },   
                                {attr: { prop: 'infoUnit', label: '单位', width:120,  } },   
                                {attr: { prop: 'quantity', label: '数量', width:120,  } },   
                                {attr: { prop: 'bookPrice', label: '账面价', width:120,  } },   
                                {attr: { prop: 'bookSum', label: '账面金额', width:120,  } },   
                                {attr: { prop: 'amortizeType', label: '摊销方式', width:120,  } },
                                {attr: { prop: 'infoRemark', label: '备注', width:120,  } }, 
                        ]
                    }
                }
            },
        //过滤器配置
        tableSelectfilter:{
             get () {
                  return {
                          formConfig:[
                            {val: '', type:'input',name:'InfoName',op:"LIKE",
                                elmentConfig:{
                                    maxlength:1000000, 
                                    minlength:0,
                                    size:"small",
                                    disabled:false,
                                    type:"text",
                                    placeholder:"请输入材料名称",
                                    icon:"",
                                    readonly:false,
                                }, 
                            },
                             {val: '', type:'input',name:'InfoModel',op:"LIKE",
                                elmentConfig:{
                                    maxlength:1000000, 
                                    minlength:0,
                                    size:"small",
                                    disabled:false,
                                    type:"text",
                                    placeholder:"请输入规格型号",
                                    icon:"",
                                    readonly:false,
                                }, 
                            },
                            ],
                            _inputArr:[
                                 {key:"OrgId",op:"EQ",value:this.getUserInfo().user.manageOrgId}
                            ],
                  }
             }
         },
        //功能配置
        tableSelectfunconfig:{
            get(){
                return {
                    funItem:{
                         loadData:{  
                            apiconf:{
                                url: '/api/services/app/turnoverReceiveOrder/GetTurnoverReceiveOrderItemPageList',
                                method: 'post',
                                data: {}
                            }, 
                         },
                    }
                }
            }
        }
    },
    methods:{
         _close(){
            this.$emit('close');
       },
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
