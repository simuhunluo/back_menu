<template>
    <el-container >
        <el-header style="background-color: #2c3e50;height: 60px;padding:0">
            <Header></Header>
        </el-header>
        <el-main style="position: absolute;top: 60px;bottom: 0px;width: 100%;" v-loading="mainLoading">
            <el-container style="height: 100%;" >
                <Aside ref="aside" class="asideDiv" :mainMenuOptions="mainMenuOptions"></Aside>
                <el-main>
                    <el-tabs id="innerTab" v-model="editableTabsValue['active-tab']" type="card" closable
                             @tab-remove="removeTab" @tab-click="clickTab" style="width: 99.8%;height: 100%;overflow-y: hidden">
                        <el-tab-pane
                                v-for="(item) in editableTabs"
                                :key="item.name"
                                :label="item.title"
                                :name="item.title" style="height: 100%;">
                                <div v-if="item.menuorigin == 'remote' && item['isShow'] == 'true'" v-html="item.content" class="remoteTabDiv"></div>
                                <div v-else class="localTabDiv">
                                    <keep-alive :include="keepAliveTagsList">
                                        <router-view v-if="item['isShow'] == 'true'"></router-view>
                                    </keep-alive>
                                </div>
                        </el-tab-pane>
                    </el-tabs>
                    <div v-show="contextMenuVisible">
                        <ul :style="{left:left+'px',top:top+'px'}" class="contextmenu">
                            <li><el-button type="text" @click="curTabReload()" size="mini">重新加载</el-button></li>
                            <li><el-button type="text" @click="closeAllTabs()" size="mini">关闭所有</el-button></li>
                            <li><el-button type="text" @click="closeOtherTabs('left')" :disabled="isDisabledCloseLeftBtnFlag" size="mini">关闭左边</el-button></li>
                            <li><el-button type="text" @click="closeOtherTabs('right')" :disabled="isDisabledCloseRightBtnFlag" size="mini">关闭右边</el-button></li>
                            <li><el-button type="text" @click="closeOtherTabs('other')" size="mini">关闭其他</el-button></li>
                        </ul>
                    </div>
                </el-main>
            </el-container>
        </el-main>
    </el-container>
</template>
<script>
    import Aside from './Aside.vue'
    import Header from "@/components/common/Header";

    export default {
        components:{
            Aside,
            Header
        },
        provide() {
            return {
                reload: this.reload
            }
        },
        data() {
            return {
                mainLoading:false,
                mainMenuOptions:[],
                editableTabsValue:this.$my_editableTabsValue,
                editableTabs:this.$my_tag_list,
                contextMenuVisible:false,
                isDisabledCloseLeftBtnFlag:true,
                isDisabledCloseRightBtnFlag:true,
                left:'',
                top:'',
                reloadKey:0,
                defaultActiveIndex:{"index":''},
            };
        },
        mounted() {
            this.bindRightClickMenu()
        },
        computed:{
            keepAliveTagsList(){
                return this.$store.getters.keepAliveTagsList
            }
        },
        watch: {
            contextMenuVisible() {
                if (this.contextMenuVisible) {
                    document.body.addEventListener("click", this.closeContextMenu);
                } else {
                    document.body.removeEventListener("click", this.closeContextMenu);
                }
            },
        },
        beforeRouteEnter (to, from, next) {
            //路由进入之前，获取数据
            next(vm => {
                vm.mainLoading = true
                vm.initMenuOptions()
            })
        },

       methods: {
           async initMenuOptions(){
               let _this = this
               await this.$axios.post("/menu/getAllMenuTreeDetailByRoleId",null,{"baseURL":'csm-base-member'})
                   .then(function (response) {
                       if(response.data.flag == 1){

                           _this.mainMenuOptions = response.data.result[0].children
                           //激活默认菜单
                           _this.mainMenu = _this.mainMenuOptions[0].title
                          // _this.$refs.aside.handleChangeForMainMenu(_this.mainMenuOptions[0].id,_this.mainMenuOptions)
                           _this.mainLoading = false
                       }else{
                           _this.$message.error(response.data.msg);
                       }
                   }).catch(function (error) {
                   console.log(error);
               });
           },
           bindRightClickMenu(){

               // 使用原生js 为单个dom绑定鼠标右击事件
               let tab_top_dom = document.body.getElementsByClassName("el-tabs__nav-scroll")
               for(let i = 0;i<tab_top_dom.length;++i){
                   tab_top_dom[i].oncontextmenu = this.openContextMenu
               }

           },

           reload(title) {

               //重新将store里的tabid设为当前页面，再使用curTabReload方法
               let currentContextTabId = title
               if(title == null ||title == ''){
                   currentContextTabId = this.editableTabsValue['active-tab']
               }
               this.$store.commit("saveCurContextTabId", currentContextTabId);
               this.curTabReload()

           },
           /**
             * 移除Tab
             * @param targetName
             */
            removeTab(targetName) {

               // 记录移除标签的index
               let targetIndex = 0;
               for(;targetIndex<this.editableTabs.length;++targetIndex){
                   if (this.editableTabs[targetIndex].title == targetName) {
                       break;
                   }
               }
               //如果移除的是当前Tab页，则激活当前页的上页或下页
               if (this.editableTabsValue['active-tab'] === targetName) {

                   let nextTab = this.editableTabs[targetIndex + 1] || this.editableTabs[targetIndex - 1];
                   if (nextTab) { //当前页并非最后一个tab页
                       //删除目标页
                       this.editableTabs.splice(targetIndex,1)
                       //调用子组件的方法，激活下一页
                       this.clickTab({"name":nextTab.title})
                   }else{ //当前页是最后一个tab页
                       //只需要 删除目标页
                       this.editableTabs.splice(targetIndex,1)
                   }
               }else{
                   ///如果移除的不是当前Tab页，只需要 删除目标页
                   this.editableTabs.splice(targetIndex,1)
               }
            },
           /*
           点击当前页
            */
           clickTab(tab) {
               let _this = this

                //查找主Tab的名称
                let childrenNode = _this.$refs.aside.mainMenuOptions.concat()
                let tempData = {"id": 0, title: '', children: childrenNode}
                let array = []
                let searchResult = {flag: false}

                this.searchMainTabName(tempData, tab.name, array, searchResult)

                //激活当前页所在的主Tab页
               _this.$refs.aside.mainHandleSelected(array[1].index,[array[1].index])
                //选中当前页所在的左侧菜单栏的
                let key = ''
                let keyPath = ''

                for (let i = 0; i <  _this.editableTabs.length; ++i) {
                    if ( _this.editableTabs[i].title == tab.name) {
                        key =  _this.editableTabs[i].key;
                        keyPath =  _this.editableTabs[i].keyPath;
                        break;
                    }
                }

               _this.$refs.aside.handleSelected(key, keyPath)

            },
           // 通过子节点找到祖宗节点
           searchMainTabName(root,target,array,searchResult){
                let _this = this
                if(searchResult.flag){
                    return
                }
                if(root.title == target){
                    array.push(root)
                    this.$set(searchResult,'flag',true)
                    return;
                }else{
                    array.push(root)
                    let children = root.children
                    if(children!= undefined && children != null) {
                        children.forEach(item => {
                            _this.searchMainTabName(item, target, array, searchResult)
                        })
                        if (!searchResult.flag) {
                            array.pop()
                        }
                    }else{
                        array.pop()
                        return;
                    }
                }
           },
            /*
            右击事件
             */
           openContextMenu(e) {
                //if(e.button == 2){  //此处不必判断鼠标点击类型
                   e.preventDefault(); //防止默认菜单弹出
                //}
               let obj = e.srcElement ? e.srcElement : e.target;

               if (obj.id) {
                   let currentContextTabId = obj.id.split("-")[1];
                   let curIndex = 0;
                   for(;curIndex<this.editableTabs.length;++curIndex){
                       if(this.editableTabs[curIndex].title == currentContextTabId){
                           break;
                       }
                   }
                   if(curIndex<=0){
                       this.isDisabledCloseLeftBtnFlag = true
                       this.isDisabledCloseRightBtnFlag = false
                   }else if(curIndex >= this.editableTabs.length-1){
                       this.isDisabledCloseLeftBtnFlag = false
                       this.isDisabledCloseRightBtnFlag = true
                   }else{
                       this.isDisabledCloseLeftBtnFlag = false
                       this.isDisabledCloseRightBtnFlag = false
                   }
                   this.contextMenuVisible = true;
                   this.$store.commit("saveCurContextTabId", currentContextTabId);

                   this.left = e.clientX;
                   this.top = 20;
               }
           },
           /*
           刷新当前页
            */
           curTabReload(){
               let currTabIndex = 0;
               for(;currTabIndex<this.editableTabs.length;++currTabIndex){
                   if(this.editableTabs[currTabIndex].title == this.$store.state.curContextTabId){
                       break;
                   }
               }
               let curTabName =  this.editableTabs[currTabIndex].name
               let new_tab_list_keepAlive = this.$store.getters.keepAliveTagsList
               new_tab_list_keepAlive = new_tab_list_keepAlive.filter((item)=>{
                   return item != curTabName
               })
               this.$store.commit('SET_KEEP_ALIVE', new_tab_list_keepAlive)
               this.$router.replace('/Home/pages/black')
               this.$nextTick(()=>{
                   this.clickTab({"name":this.editableTabs[currTabIndex].title})
                   this.$notify({
                       title: this.$t('message.tip'),
                       message: "刷新成功（如显示空白，请重试刷新！）",
                       position: 'bottom-right'
                   });
               })
           },
           // 关闭所有标签页
           closeAllTabs() {
               //删除所有tab标签
               this.editableTabs.splice(0,this.$my_tag_list.length)
               //调用子组件的方法，设置默认选中
               // this.$refs.aside[0].handleSelected("3",["3"]);
               this.closeContextMenu()
           },

           // 关闭其它标签页
           closeOtherTabs(par) {

               let currTabIndex = 0;
               for(;currTabIndex<this.editableTabs.length;++currTabIndex){
                   if(this.editableTabs[currTabIndex].title == this.$store.state.curContextTabId){
                       break;
                   }
               }
               let curTabTitle = this.editableTabs[currTabIndex].title
               switch(par){
                   case "left":{
                       //删除左侧tab标签
                       this.editableTabs.splice(0,currTabIndex)
                       break;
                   }
                   case "right":{
                       //删除右侧tab标签
                       this.editableTabs.splice(currTabIndex+1,this.editableTabs.length)
                       break;
                   }
                   case "other":{
                       //删除其他所有tab标签
                       this.editableTabs.splice(0,currTabIndex)
                       this.editableTabs.splice(currTabIndex+1,this.editableTabs.length)
                       break;
                   }
               }
               this.clickTab({"name":curTabTitle})
               this.closeContextMenu()
           },
           // 关闭contextMenu
           closeContextMenu() {
               this.contextMenuVisible = false;
           },
        }
    }
</script>
<style>
    div.el-tabs__header.is-top {
        padding: 0;
        position: relative;
        margin: 0 0 5px;
    }
    .contextmenu {
        width: 100px;
        margin: 0;
        border: 1px solid #ccc;
        background: #fff;
        z-index: 3000;
        position: absolute;
        list-style-type: none;
        padding: 5px 0;
        border-radius: 4px;
        font-size: 14px;
        color: #333;
        box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, 0.2);
    }
    .contextmenu li {
        margin: 0;
        padding: 0px 22px;

    }
    .contextmenu li:hover {
        background: #f2f2f2;
        cursor: pointer;
    }
    .contextmenu li button{
       color: #2c3e50;
    }
    .remoteTabDiv {
        height: 98%;
        margin: 0;
        width: 99.4%;
    }
    .asideDiv{
        background-color:#223142;
        height: 100%;
        overflow-y: auto;
        overflow-x: hidden;
    }
    .localTabDiv {
        height: 98%;
        width:99.4%;
        padding-top: 5px;
        padding-left: 10px;
        overflow-y: auto;
    }
    .el-tabs__item {
        padding: 0 20px;
        height: 30px;
        box-sizing: border-box;
        line-height: 30px;
        display: inline-block;
        list-style: none;
        font-size: 14px;
        font-weight: 500;
        color: #303133;
        position: relative;
    }
    .el-tabs--border-card>.el-tabs__content {
        padding: 0px;
        overflow-y: auto;
        overflow-x: hidden;
        position: absolute;
        width: 100%;
        top: 29px;
        bottom: 0px;

    }
    #innerTab .el-tabs__content{
        height: 100%;
        box-sizing: border-box;
        overflow-y: auto;
    }
    .el-main {
        display: block;
        flex: 1;
        flex-basis: auto;
        overflow: auto;
        box-sizing: border-box;
        padding: 0px;
    }
    .el-tab-pane {
        position: absolute;
        width: 100%;
        top: 0px;
        bottom: 0px;
    }
</style>