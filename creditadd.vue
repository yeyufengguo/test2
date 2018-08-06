<style lang="less">
   @import '../../styles/upload.less';
</style>

<template>
 <div>
        <Row>
            <Card>
                <Form :label-width="80" ref="creditForm" :model="creditForm" :rule="creditFormRule">
                    <FormItem label="所属类型" prop="type">
                        <Select placeholder="所属类型" v-model="creditForm.type" style="width: 300px">
                            <Option :value="1">个人</Option>
                            <Option :value="2">企业</Option>
                        </Select>
                    </FormItem>
                    <FormItem label="联系电话">
                        <Select
                            :label="mobile"
                            v-model="mobile"
                            filterable
                            remote
                            placeholder="请输入客户联系电话"
                            style="width: 300px"
                            :remote-method="customerSearch"
                            @on-change="selectCus"
                            clearable
                            label-in-value
                            :loading="loading1">
                            <Option v-for="(option, index) in customerList" :value="option.mobile" :label="index" :key="index">{{option.selectdata}}</Option>
                        </Select>
                         <span style="margin-left:15px;">没有查询到客户姓名？点我<Button type="text" @click="alertAdd" style="padding-left:5px;color:#ed3f14">添加</Button></span>
                    </FormItem>
                    <FormItem label="客户姓名">
                        <Input type="text" readonly v-model="creditForm.customer_name" style="width: 300px"></Input>
                    </FormItem>
                    <FormItem label="证件号码" v-if="creditForm.certdata.length>0">
                        <Tag v-for="(item, index) in creditForm.certdata" :key="index" class="credit-tag">{{item.certcode}}（{{item.certname}}）</Tag>
                        <Button type="ghost" @click="alertAddCard">添加</Button>
                    </FormItem>
                    <FormItem label="理财经理">
                        <Input type="text" readonly style="width: 300px" v-model="financing_manager_name"></Input>
                    </FormItem>
                    <FormItem label="授权材料">
                        <div class="upload-list" v-for="(item, index) in uploadList" :key="index">
                            <template v-if="item.status === 'finished' || !item.status">
                                <img :src="item.url?item.url:item.response.data.url">
                                <div class="upload-list-cover">
                                    <Icon type="ios-eye-outline" @click.native="handleView(item)"></Icon>
                                    <Icon type="ios-trash-outline" @click.native="handleImgRemove(item)"></Icon>
                                </div>
                            </template>
                            <template v-else>
                                <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
                            </template>
                        </div>
                        <Upload
                            ref="upload"
                            :show-upload-list="false"
                            :default-file-list="defaultList"
                            :on-success="handleSuccess"
                            :format="['jpg','jpeg','png']"
                            :max-size="2048"
                            :on-format-error="handleFormatError"
                            :on-exceeded-size="handleMaxSize"
                            :before-upload="handleBeforeUpload"
                            multiple
                            type="drag"
                            name="pic"
                            :action="uploadUrl"
                            style="display: inline-block;width:58px;">
                            <div style="width: 58px;height:58px;line-height: 58px;">
                                <Icon type="camera" size="20"></Icon>
                            </div>
                        </Upload>
                        <Modal title="查看图片" v-model="visible">
                            <img :src="url" v-if="visible" style="width: 100%">
                        </Modal>
                        <span style="color:#ed3f14;margin-left:15px;">请上传jpg,jpeg,png,png格式图片，限6张</span>
                    </FormItem>
                    <FormItem label="备注说明">
                        <Input type="textarea" :rows="4" style="width: 300px" v-model="creditForm.remark_base"></Input>
                    </FormItem>
                    <FormItem>
                        <Button type="primary" @click="submitApplayInfo('creditForm')">确定</Button>
                        <Button type="ghost" style="margin-left: 8px" @click="closeCurrentPage">取消</Button>
                    </FormItem>
                </Form>
            </Card>
        </Row>
        <Modal v-model="modalSetting.show" :mask-closable="false" width="668" :styles="{top: '100px'}" @on-visible-change="doCancel">
            <p slot="header" style="color:#2d8cf0;">
                <Icon type="information-circled"></Icon>
                <span>新增客户</span>
            </p>
            <Form ref="formItem" :model="formItem" :rules="itemRule" :label-width="80">
                <FormItem label="所属类型" prop="ctype">
                    <Select placeholder="所属类型" v-model="formItem.ctype" @on-change="changeCtype">
                        <Option :value="1">个人</Option>
                        <Option :value="2">企业</Option>
                    </Select>
                </FormItem>
                <FormItem label="客户姓名" prop="cname">
                    <Row>
                        <Col span="18">
                            <Input type="text" v-model="formItem.cname"></Input> 
                        </Col>
                        <Col span="5" offset="1">
                        <ButtonGroup>
                            <Button :type="formItem.gender==1?'primary':'ghost'" @click="setGenter(1)">先生</Button>
                            <Button :type="formItem.gender==2?'primary':'ghost'" @click="setGenter(2)">女士</Button>
                        </ButtonGroup>
                        </Col>
                    </Row>
                </FormItem>
                <FormItem label="联系电话" prop="mobile">
                   <Input type="text" v-model="formItem.mobile"></Input>
                </FormItem>
                <FormItem label="证件号码" 
                    v-for="(item, index) in formItem.certdata"
                    :key="index"
                    :prop="'certdata.' + index + '.certcode'"
                    :rules="{required: true, message: '证件号码不能为空', trigger: 'blur'}">
                    <Row>
                        <Col span="20">
                            <Input v-model="item.certcode">
                            <Select v-model="item.certtype" slot="prepend" style="width: 150px">
                                <Option v-for="(option, index) in certtypeList" :value="option.code" :key="index">{{option.valname}}</Option>
                            </Select>
                            </Input>
                        </Col>
                        <Col span="3" offset="1">
                            <Button type="ghost" @click="handleAdd('formItem')" v-if="index==0">添加</Button>
                            <Button type="ghost" @click="handleRemove(index,'formItem')" v-if="index!==0">删除</Button>
                        </Col>
                    </Row>
                </FormItem>
                <FormItem label="理财经理" prop="financing_manager_id">
                    <Select
                        v-model.number="formItem.financing_manager_id"
                        filterable
                        remote
                        clearable
                        placeholder="请输入理财经理姓名"
                        :remote-method="managerSearch"
                        :loading="loading1">
                        <Option v-for="(option, index) in managerList" :value="option.id" :key="index">{{option.managername}}（{{option.deptname}}）</Option>
                    </Select>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="text" @click="cancel('formItem')" style="margin-right: 8px">取消</Button>
                <Button type="primary" @click="addCustomer('formItem')" :loading="modalSetting.loading">确定</Button>
            </div>
        </Modal>
        <Modal v-model="cardmodalSetting.show" :mask-closable="false" width="668" :styles="{top: '100px'}">
            <p slot="header" style="color:#2d8cf0;">
                <Icon type="information-circled"></Icon>
                <span>添加证件</span>
            </p>
            <Form ref="cardForm" :model="cardForm" :label-width="100">
                <FormItem label="客户姓名：">
                      {{creditForm.customer_name}}
                </FormItem>
                <FormItem label="理财经理：">
                    {{financing_manager_name}}
                </FormItem>
                <FormItem label="证件号码：" 
                v-for="(item, index) in cardForm.certdata"
                :key="index"
                :prop="'certdata.' + index + '.certcode'"
                :rules="{required: true, message: '证件号码不能为空', trigger: 'blur'}">
                    <Row>
                        <Col span="20">
                            <Input v-model="item.certcode">
                            <Select v-model="item.certtype" slot="prepend" style="width: 150px">
                                <Option v-for="(option, index) in certtypeList" :value="option.code" :key="index">{{option.valname}}</Option>
                            </Select>
                            </Input>
                        </Col>
                        <Col span="3" offset="1">
                            <Button type="ghost" @click="handleAdd('cardForm')" v-if="index==0">添加</Button>
                            <Button type="ghost" @click="handleRemove(index,'cardForm')" v-if="index!==0">删除</Button>
                        </Col>
                    </Row>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="text" @click="cancel('cardForm')" style="margin-right: 8px">取消</Button>
                <Button type="primary" @click="addCard('cardForm')" :loading="cardmodalSetting.loading">确定</Button>
            </div>
        </Modal>
    </div>
</template>
<style scoped>
.credit-tag{
    height:32px !important;
    line-height:32px !important;
}
</style>

<script>
    import config from '../../../build/config';
    import axios from 'axios';
    import Util from '@/libs/util';
    import { validateMobile } from '@/libs/validate';

    export default {
        name: 'creditadd',
        data () {
            const validatemobile = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('联系电话不能为空'));
                } else {
                    if (validateMobile(value)) {
                        callback();
                    } else {
                        callback(new Error('联系电话格式不正确'));
                    }
                }
            };
            return {
                confirmSub: false,
                isprimary: false,
                uploadUrl: '',
                url: '',
                saveLoading: false,
                imgName: '',
                visible: false,
                mobile: '', // 编辑时下拉选中项
                uploadList: [],
                defaultList: [],
                formItem: {
                    ctype: 1,
                    cname: '',
                    mobile: '',
                    gender: 1,
                    certdata: [
                        {
                            certtype: '',
                            certcode: ''
                        }
                    ],
                    financing_manager_id: ''
                },
                loading1: false,
                options1: [],
                customerList: [],
                managerList: [],
                financing_manager_name: '',
                // 证件
                certtypeList: [],
                // 个人、企业证件
                allcerttypeList: [],
                modalSetting: {
                    show: false,
                    loading: false,
                    index: 0
                },
                cardmodalSetting: {
                    show: false,
                    loading: false,
                    index: 0
                },
                index: 1,
                creditForm: {
                    id: '',
                    type: 1,
                    customer_name: '',
                    mobile: '',
                    certdata: [],
                    financing_dept_id: '',
                    financing_manager_id: '',
                    picture: [],
                    remark_base: ''
                },
                cardForm: {
                    id: '',
                    certdata: [
                        {
                            certtype: '',
                            certcode: ''
                        }
                    ],
                    index: 1
                },
                creditFormRule: {
                    type: [
                        {type: 'number', required: true, message: '所属类型不能为空', trigger: 'change'}
                    ]
                },
                itemRule: {
                    ctype: [
                        {type: 'number', required: true, message: '所属类型不能为空', trigger: 'change'}
                    ],
                    cname: [
                        {required: true, message: '客户姓名不能为空', trigger: 'blur'}
                    ],
                    mobile: [
                        {required: true, validator: validatemobile, trigger: 'blur'}
                    ],
                    financing_manager_id: [
                        { type: 'number', required: true, message: '理财经理不能为空', trigger: 'change' }
                    ]
                }
            };
        },
        created () {
            this.getDictionary();
            this.init();
            let id = this.$route.query.id;
            if (id) {
                this.getCredit(id);
                let item = {
                    vm: this,
                    title: '编辑征信申请'
                };
                this.$store.commit('setTagstitle', item);
            } else {
                let item = {
                    vm: this,
                    title: '新增征信申请'
                };
                this.$store.commit('setTagstitle', item);
            }
        },
        methods: {
            confirmSubmit () {
                this.confirmSub = true;
            },
            submitApplayInfo (name) {
                let vm = this;
                vm.$refs[name].validate((valid) => {
                    if (valid) {
                        vm.saveLoading = true;
                        let url = 'credit/addcredit';
                        if (vm.creditForm.id) {
                            url = 'credit/editcredit';
                        };
                        axios.post(url, vm.creditForm).then(function (response) {
                            if (response.data.code === 1) {
                                vm.$Message.success(response.data.msg);
                                Util.closeCurrentPage(vm);
                            } else {
                                vm.$Message.error(response.data.msg);
                            }
                        });
                    }
                });
            },
            addCustomer (name) {
                let vm = this;
                vm.$refs[name].validate((valid) => {
                    if (valid) {
                        vm.modalSetting.loading = true;
                        axios.post('Customer/addcreditCustomer', vm.formItem).then(function (response) {
                            vm.modalSetting.loading = false;
                            if (response.data.code === 1) {
                                vm.modalSetting.show = false;
                                vm.$Message.success(response.data.msg);
                            } else {
                                vm.$Message.error(response.data.msg);
                            }
                        });
                    }
                });
            },
            addCard (name) {
                let vm = this;
                vm.$refs[name].validate((valid) => {
                    if (valid) {
                        vm.cardmodalSetting.loading = true;
                        axios.post('customer/addcard', {id: vm.creditForm.id, certdata: vm.cardForm.certdata}).then(function (response) {
                            vm.cardmodalSetting.loading = false;
                            if (response.data.code === 1) {
                                vm.$Message.success(response.data.msg);
                                vm.cardmodalSetting.show = false;
                                if (response.data.data) {
                                    vm.creditForm.certdata = vm.creditForm.certdata.concat(response.data.data);
                                }
                                vm.cardForm.certdata = [{certtype: '', certcode: ''}];
                            } else {
                                vm.$Message.error(response.data.msg);
                            }
                        });
                    }
                });
            },
            init () {
                this.uploadUrl = config.baseUrl + 'index/fileUpload';
            },
            handleFormatError (file) {
                this.$Notice.warning({
                    title: '文件类型不合法',
                    desc: file.name + '的文件类型不正确，请上传jpg或者png图片。'
                });
            },
            handleImgRemove (file) {
                const fileList = this.$refs.upload.fileList;
                let index = fileList.indexOf(file);
                this.$refs.upload.fileList.splice(index, 1);
                this.creditForm.picture.splice(index, 1);
            },
            handleSuccess (response, file, fileList) {
                if (response.code === 1) {
                    this.$Message.success(response.msg);
                    this.creditForm.picture.push(response.data.id);
                    this.uploadList = fileList;
                } else {
                    this.$Message.error(response.msg);
                }
            },
            handleMaxSize (file) {
                this.$Notice.warning({
                    title: '文件大小不合法',
                    desc: file.name + '太大啦请上传小于5M的文件。'
                });
            },
            handleBeforeUpload () {
                const check = this.uploadList.length < 6;
                if (!check) {
                    this.$Notice.warning({
                        title: '最多只能上传 6 张图片。'
                    });
                }
                return check;
            },
            handleView (item) {
                this.imgName = item.name;
                this.url = item.url ? item.url : item.response.data.url;
                this.visible = true;
            },
            // 客户搜素
            customerSearch (query) {
                let vm = this;
                if (query !== '') {
                    vm.loading1 = true;
                    vm.customerList = [];
                    axios.get('customer/getcusinfo', {params: {'mobile': query, 'type': vm.creditForm.type}}).then(function (response) {
                        vm.loading1 = false;
                        let res = response.data;
                        if (res.code === 1) {
                            vm.customerList = res.data;
                        } else {
                            vm.customerList = [];
                        }
                    });
                } else {
                    vm.customerList = [];
                }
            },
            // 理财经理
            managerSearch (query) {
                let vm = this;
                if (query !== '') {
                    this.loading1 = true;
                    axios.get('systemuser/managerlist', {params: {'name': query}}).then(function (response) {
                        vm.loading1 = false;
                        let res = response.data;
                        if (res.code === 1) {
                            vm.managerList = res.data;
                        } else {
                            vm.managerList = [];
                        }
                    });
                } else {
                    vm.managerList = [];
                }
            },
            alertAdd () {
                this.modalSetting.show = true;
                this.formItem.certdata = [{certtype: '', certcode: ''}];
            },
            cancel (item) {
                if (item === 'formItem') {
                    this.modalSetting.show = false;
                };
                if (item === 'cardForm') {
                    this.cardmodalSetting.show = false;
                }
            },
            doCancel (data) {
                if (!data) {
                    this.formItem.id = 0;
                    this.$refs['formItem'].resetFields();
                    this.modalSetting.loading = false;
                    this.modalSetting.index = 0;
                }
            },
            handleAdd (item) {
                if (item === 'formItem') {
                    this.index++;
                    this.formItem.certdata.push({
                        certtype: '',
                        certcode: ''
                    });
                };
                if (item === 'cardForm') {
                    this.cardForm.index++;
                    this.cardForm.certdata.push({
                        certtype: '',
                        certcode: ''
                    });
                }
            },
            handleRemove (index, item) {
                if (item === 'formItem') {
                    this.formItem.certdata.splice(index, 1);
                };
                if (item === 'cardForm') {
                    this.cardForm.certdata.splice(index, 1);
                }
            },
            setGenter (index) {
                this.formItem.gender = index;
            },
            // 添加证件
            alertAddCard () {
                this.cardmodalSetting.show = true;
            },
            selectCus (option) {
                let index = option.label;
                let label = option.value; // 手机号码
                let id = this.$route.query.id;
                if (id && label && !index) { // 编辑申请
                    return true;
                }
                if (index === '') {
                    this.creditForm.mobile = '';
                    this.creditForm.customer_name = '';
                    this.financing_manager_name = '';
                    this.creditForm.financing_manager_id = '';
                    this.creditForm.financing_dept_id = '';
                    this.creditForm.certdata = [];
                    this.cardForm.id = '';
                    return true;
                };
                let item = this.customerList[index];
                this.creditForm.mobile = item.mobile;
                this.creditForm.customer_name = item.cname;
                this.financing_manager_name = item.financing_manager_name;
                this.creditForm.financing_manager_id = item.financing_manager_id;
                this.creditForm.financing_dept_id = item.financing_dept_id;
                this.creditForm.certdata = item.certdata;
                this.cardForm.id = item.id;
            },
            closeCurrentPage () {
                Util.closeCurrentPage(this);
            },
            getCredit (id) {
                let vm = this;
                axios.get('credit/getcreditinfo', {params: {'id': id}}).then(function (response) {
                    let res = response.data;
                    if (res.code === 1) {
                        vm.creditForm.certdata = res.data.certdata;
                        vm.creditForm.remark_base = res.data.remark_base;
                        vm.creditForm.customer_name = res.data.customer_name;
                        vm.creditForm.mobile = res.data.mobile;
                        vm.mobile = res.data.mobile;
                        vm.creditForm.financing_manager_id = res.data.financing_manager_id;
                        vm.creditForm.id = res.data.id;
                        vm.creditForm.type = res.data.type;
                        vm.creditForm.financing_dept_id = res.data.financing_dept_id;
                        vm.defaultList = res.data.auth_picurl;
                        res.data.auth_picurl.forEach(item => vm.creditForm.picture.push(item.id));
                        vm.uploadList = res.data.auth_picurl;
                        vm.financing_manager_name = res.data.financing_manager_name;
                    } else {
                        vm.$Message.error(res.msg);
                    }
                });
            },
            // 选择所属类型
            changeCtype () {
                if (this.formItem.ctype === 1) {
                    this.certtypeList = this.allcerttypeList.certtype;
                };
                if (this.formItem.ctype === 2) {
                    this.certtypeList = this.allcerttypeList.enterprice_certtype;
                };
            },
            // 数据字典
            getDictionary () {
                let vm = this;
                axios.post('dictionary/getdictionarybytype', {'type': ['CERTTYPE', 'ENTERPRICE_CERTTYPE']}).then(function (response) {
                    let res = response.data;
                    if (res.code === 1) {
                        vm.allcerttypeList = res.data;
                        if (vm.formItem.ctype === 1) {
                            vm.certtypeList = res.data.certtype;
                        };
                        if (vm.formItem.ctype === 2) {
                            vm.certtypeList = res.data.enterprice_certtype;
                        };
                    } else {
                        if (res.code === -14) {
                            vm.$store.commit('logout', vm);
                            vm.$router.push({
                                name: 'login'
                            });
                        } else {
                            vm.$Message.error(res.msg);
                        }
                    }
                });
            }
        }
    };
</script>
