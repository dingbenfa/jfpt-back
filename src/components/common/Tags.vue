<template>
    <div class="tags" :class="{'content-collapse':collapse}">
        <!-- 折叠按钮 -->
        <div class="dbfadmin-item-box dbfadmin-collapse" @click="collapseChage">
            <i class="el-icon-menu"></i>
        </div>
        <div class="dbfadmin-item-box dbfadmin-refresh" @click="refreshPage">
            <i class="el-icon-refresh"></i>
        </div>
        <div class="page-tags">
            <el-tabs v-model="editableTabsValue" type="border-card" @tab-click="onRoutes" @tab-remove="removeTab">
                <el-tab-pane v-for="(item, index) in editableTabs" :key="index" :label="item.title" :name="item.name" :closable="item.closable">
                </el-tab-pane>
            </el-tabs>
        </div>
        <div class="dbfadmin-item-box dbfadmin-tags-select">
            <el-dropdown @command="handleTags">
                <span class="el-dropdown-link">
                    <i class="el-icon-arrow-down"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item command="now">关闭当前标签页</el-dropdown-item>
                    <el-dropdown-item command="other">关闭其它标签页</el-dropdown-item>
                    <el-dropdown-item command="all">关闭全部标签页</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </div>
    </div>
</template>

<script>
    import bus from './bus';
    export default {
        data() {
            return {
                editableTabsValue: 'dashboard',
                editableTabs: [{
                    title: '系统首页',
                    name: 'dashboard',
                    path: '/dashboard',
                    content: '',
                    closable: false
                }],
                collapse: false
            }
        },
        methods: {
            addTab(targetTitle,targetName,targetPath) {
                let newTabName = targetName;
                this.editableTabs.push({
                    title: targetTitle,
                    name: newTabName,
                    path: targetPath,
                    content: '',
                    closable: true
                });
                this.editableTabsValue = newTabName;
            },
            removeTab(targetName) {
                let tabs = this.editableTabs;
                let activeName = this.editableTabsValue;
                let targetRoutes = new Object();
                if (activeName === targetName) {
                    tabs.forEach((tab, index) => {
                        if (tab.name === targetName) {
                            let nextTab = tabs[index + 1] || tabs[index - 1];
                            if (nextTab) {
                                activeName = nextTab.name;
                            }
                        }
                    });
                }
                
                this.editableTabsValue = activeName;
                this.editableTabs = tabs.filter(tab => tab.name !== targetName);
                
                targetRoutes.name = activeName;
                this.onRoutes(targetRoutes);
            },
            selectTab(targetTitle){
                let tabs = this.editableTabs;
                let activeName = this.editableTabsValue;
                tabs.forEach((tab, index) => {
                    if (tab.title === targetTitle) {
                        activeName = tab.name;
                    }
                });
                
                this.editableTabsValue = activeName;
            },
            isHasTheTitle(targetTitle){
                let tabs = this.editableTabs;
                let n = false;
                tabs.forEach((tab, index) => {
                    if (tab.title === targetTitle) {
                        n = true;
                        return false;
                    }
                });
                return n;
            },
            isClosable(targetName){
                let tabs = this.editableTabs;
                let activeName = this.editableTabsValue;
                let n = false;
                tabs.forEach((tab, index) => {
                    if (tab.name === activeName && tab.closable) {
                        n = true;
                        return false;
                    }
                });

                return n;
            },
            getThePath(targetName){
                let tabs = this.editableTabs;
                let thePath = this.editableTabsValue;
                tabs.forEach((tab, index) => {
                    if (tab.name === targetName) {
                        thePath = tab.path;
                        return false;
                    }
                });

                return thePath;
            },
            onRoutes(target){
                let activePath = this.getThePath(target.name);
                this.$router.push(activePath);
            },
            // 侧边栏折叠
            collapseChage(){
                this.collapse = !this.collapse;
                bus.$emit('collapse', this.collapse);
            },
            //关闭当前标签
            closeNow(){
                this.isClosable(this.editableTabsValue)===true? this.removeTab(this.editableTabsValue): false
            },
            // 关闭全部标签
            closeAll(){
                this.editableTabs = [{
                    title: '系统首页',
                    name: 'dashboard',
                    path: '/dashboard',
                    content: '',
                    closable: false
                }];
                this.$router.push('/');
            },
            // 关闭其他标签
            closeOther(){
                const curItem = this.editableTabs.filter(item => {
                    if(item.closable){
                        return item.path === this.$route.fullPath;
                    }else{
                        return true;
                    }
                })
                this.editableTabs = curItem;
            },
            // 设置标签
            setTags(route){
                let tagTitle = route.meta.title;
                let tagName = route.matched[1].components.default.name;
                let tagPath = route.fullPath;

                this.isHasTheTitle(tagTitle)===true? this.selectTab(tagTitle):this.addTab(tagTitle,tagName,tagPath);

                bus.$emit('tags', this.editableTabs);
            },
            //刷新
            refreshPage () {
                bus.$emit('reload', this.editableTabs);
            },
            handleTags(command){
                switch(command){
                    case "now": 
                        this.closeNow();
                        break;
                    case "other": 
                        this.closeOther();
                        break;
                    case "all": 
                        this.closeAll();
                        break;
                    default: this.closeAll();
                }
            }
        },
        computed: {
            showTags() {
                return this.editableTabs.length > 0;
            }
        },
        watch:{
            $route(newValue, oldValue){
                this.setTags(newValue);
            }
        },
        created(){
            bus.$on('collapse', msg => {
                this.collapse = msg;
            })
            this.setTags(this.$route);
        },
        mounted(){
            if(document.body.clientWidth < 1500){
                this.collapseChage();
            }
        }
    }

</script>


<style>
    .tags {
        position: absolute;
        top: 70px;
        left: 250px;
        right: 0;
        height: 40px;
        line-height: 40px;
        overflow: hidden;
        background-color: #fff;
        box-sizing: border-box;
        box-shadow: 0 1px 2px 0 rgba(0, 0, 0, .1);
        -webkit-transition: left .3s ease-in-out;
        transition: left .3s ease-in-out;
    }
    .page-tags{
        position: relative;
        height: 40px;
        overflow: hidden;
        margin: 0 40px 0 80px;
        background-color: #fff;
        box-sizing: border-box;
    }
    .dbfadmin-item-box{
        position: absolute;
        top: 0;
        width: 40px;
        line-height: 42px;
        height: 100%;
        text-align: center;
        cursor: pointer;
        transition: all .3s;
        -webkit-transition: all .3s;
        box-sizing: border-box;
        border-right: 1px solid #efefef;
        color: #6d6d6d;
    }
    .dbfadmin-item-box i{
        font-weight: bold;
    }
    .dbfadmin-collapse{
        left: 0;
    }
    .dbfadmin-refresh{
        left: 40px;
    }
    .dbfadmin-tags-select{
        right: 0;
        border-left: 1px solid #efefef;
    }
    .dbfadmin-tags-select .el-dropdown{
        width: 40px;
        height: 40px;
    }
    .dbfadmin-tags-select .el-dropdown-link{
        display: inline-block;
        width: 40px;
        height: 40px;
    }
    .dbfadmin-item-box:hover{
        color: #358edc;
        background-color: #f2f2f2;
    }

    .el-tabs__nav-next, .el-tabs__nav-prev {
        position: absolute;
        cursor: pointer;
        line-height: 43px;
        font-size: 17px;
        color: #909399;
        font-weight: bold;
        width: 40px;
        text-align: center;
    }
    .el-tabs__nav-prev{
        border-right: 1px solid #efefef;
    }
    .el-tabs__nav-next{
        border-left: 1px solid #efefef;
    }
    .el-tabs__nav-prev:hover{
        background-color: #f2f2f2;
    }
    .el-tabs__nav-next:hover{
        background-color: #f2f2f2;
    }
    .el-tabs__nav-wrap.is-scrollable {
        padding: 0px 40px;
    }
    .el-tabs--border-card{
        border: none;
    }
    .el-tabs__item {
        position: relative;
        border: none;
    }
    .el-tabs__item:hover{
        color: #000 !important;
    }
    .el-tabs__item::before{
        content: '';
        position: absolute;
        top: 0;
        right: 0;
        width: 0;
        height: 40px;
    }
    .el-tabs__item::after{
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        width: 0;
        height: 2px;
        border-radius: 0;
        background-color: #292B34;
        transition: all .3s;
        -webkit-transition: all .3s;
    }
    .el-tabs__item:not(.is-active):hover {
        background: #f8f8f8;
    }
    .el-tabs__item:not(.is-active):hover:after{
        width: 100%;
    }
    .el-tabs--border-card>.el-tabs__header .el-tabs__item.is-active{
        background-color: #f6f6f6;
        border: 1px solid transparent;
    }
    .el-tabs__item.is-active::after {
        width: 100%;
        border: none;
        height: 2px;
        background-color: #292B34;
    }
    .el-tabs__item .el-icon-close:hover{
        color: #fff !important;
        background-color: #ff5722;
    }
    .tags-fiexd .tags-li-title{
        margin-right: 0px;
    }
    .el-tabs--border-card>.el-tabs__header{
        background-color: #fff;
        border-bottom: none;
    }
    .el-tabs__content{
        padding: 0;
    }
    .el-tabs--border-card>.el-tabs__header .el-tabs__item.is-active{
        color: #000;
        background-color: #f6f6f6;
    }
    .el-tabs--border-card>.el-tabs__header .el-tabs__item {
        margin: 0 -1px 0;
         border-right: 1px solid #efefef;
    }

</style>
