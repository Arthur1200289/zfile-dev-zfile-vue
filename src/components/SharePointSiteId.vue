<template>
    <el-row :gutter="20">
        <el-col :span="12" :offset="6">
            <h2>在线获取 SharePoint SiteId 小工具</h2>
            <el-form ref="form" :rules="rules" :model="form">
                <el-form-item label="版本" class="box animate__animated animate__fadeInUp">
                    <el-radio v-model="form.type" @change="changeType" label="Standard">国际版</el-radio>
                    <el-radio v-model="form.type" @change="changeType" label="China">世纪互联</el-radio>
                </el-form-item>
                <el-form-item label="AccessToken" prop="accessToken" class="box animate__animated animate__fadeInUp">
                    <el-input v-model="form.accessToken" placeholder="请输入 AccessToken"></el-input>
                    <el-link target="_blank" v-show="form.type === 'Standard'" icon="el-icon-link" :href="$http.defaults.baseURL + '/onedrive/authorize'">前往获取令牌 (国际版)</el-link>
                    <el-link target="_blank" v-show="form.type === 'China'" icon="el-icon-link" :href="$http.defaults.baseURL + '/onedrive/china-authorize'">前往获取令牌 (世纪互联)</el-link>
                </el-form-item>
                <el-form-item label="SharePoint 域名" prop="domainPrefix" class="box animate__animated animate__fadeInUp">
                    <el-input v-model="form.domainPrefix" placeholder="请输入 SharePoint 域名">
                        <template slot="append">.sharepoint.{{ form.domainType }}</template>
                    </el-input>
                </el-form-item>
                <el-form-item label="站点名称" prop="domainPrefix" class="box animate__animated animate__fadeInUp">
                    <template slot="label">
                        <span>站点名称</span>
                        <span class="zfile-word-aux">（网址上域名后面的 /sites/xxx 或/teams/xxx）</span>
                    </template>
                    <el-input v-model="form.siteName" placeholder="请输入站点名称"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" size="small" @click="submitForm('form')">获取 SiteId</el-button>
                </el-form-item>

                <el-form-item label="SiteID" v-if="siteId" class="box animate__animated animate__fadeInUp">
                    <el-input type="small" v-model="siteId">
                        <el-tooltip slot="append" class="item" effect="dark" content="复制" placement="bottom">
                            <el-button @click="copyText(siteId)" type="small" style="font-size: 20px" icon="el-icon-copy-document"></el-button>
                        </el-tooltip>
                    </el-input>
                </el-form-item>
            </el-form>
        </el-col>

        <el-input placeholder="请输入内容" v-model="input3" class="input-with-select">
            <el-select v-model="select" slot="prepend" placeholder="请选择">
                <el-option label="餐厅名" value="1"></el-option>
                <el-option label="订单号" value="2"></el-option>
                <el-option label="用户电话" value="3"></el-option>
            </el-select>
            <el-button slot="append" icon="el-icon-search"></el-button>
        </el-input>
    </el-row>
</template>

<script>
export default {
    name: "SharePointSiteId",
    data() {
        return {
            input1: '',
            input2: '',
            input3: '',
            select: '',
            form: {
                type: 'Standard',
                accessToken: '',
                domainPrefix: '',
                siteName: '/sites/xxx',
                domainType: 'com'
            },
            siteId: '',
            rules: {
                accessToken: [
                    {required: true, message: 'AccessToken 不能为空', trigger: 'change'},
                ],
                domainPrefix: [
                    {required: true, message: 'SharePoint 域名不能为空', trigger: 'change'},
                ],
                siteName: [
                    {required: true, message: '请输入管理员密码', trigger: 'change'},
                ]
            },
            loading: false
        }
    },
    watch: {
        'form.type'() {
            if (!!this.form.accessToken) {
                this.getDomainPrefix();
            }
        },
        'form.accessToken'() {
            if (!!this.form.accessToken) {
                this.getDomainPrefix();
            }
        }
    },
    methods: {
        getDomainPrefix() {
            this.$http.post('/sharepoint/getDomainPrefix', this.form).then((response) => {
                if (response.data.code === 0) {
                    this.form.domainPrefix = response.data.data;
                }
            })
        },
        copyText(text) {
            this.$copyText(text).then(() => {
                this.$message.success('复制成功');
            }, () => {
                this.$message.error('复制失败');
            });
        },
        changeType() {
            debugger;
            if (this.form.type === 'Standard') {
                this.form.domainType = 'com';
            } else if (this.form.type === 'China') {
                this.form.domainType = 'cn';
            }
        },
        submitForm(formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    this.loading = true;
                    this.$http.post('/sharepoint/getSiteId', this.form).then((response) => {
                        if (response.data.code === 0) {
                            this.siteId = response.data.data;
                        } else {
                            this.$message.error('获取失败');
                        }

                        this.loading = false;
                    })
                } else {
                    this.loading = false;
                    return false;
                }
            });
        }
    }
}
</script>

<style scoped>

</style>