<template>
    <el-row>
        <el-col :xs="2" :sm="4" :md="4" :lg="2" :xl="2" class="hidden-xs-only">
            <div class="logo">后台管理系统</div>
        </el-col>
        <el-col :xs="1" :sm="7" :md="8" :lg="14" :xl="14" >
            <div style="min-height: 36px;"></div>
        </el-col>
        <el-col :xs="8" :sm="4" :md="4" :lg="2" :xl="2">

            <!-- 全屏显示 -->
            <div  class="btn-fullscreen " @click="handleFullScreen">
                <el-tooltip effect="dark" :content="fullscreen?`取消全屏`:`全屏`" placement="bottom" class="hidden-md-and-down">
                    <i class="el-icon-rank"></i>
                </el-tooltip>
            </div>
            <!-- 消息中心 -->
            <div class="btn-bell">
                <el-tooltip
                        effect="dark"
                        :content="message?`有${message}条未读消息`:`消息中心`"
                        placement="bottom"
                >
                    <div style="height: 50px">
                        <i class="el-icon-bell"></i>
                        <span class="btn-bell-badge" v-if="message"></span>
                    </div>
                </el-tooltip>
            </div>
            <!-- 刷新Tab -->
            <div  class="btn-refresh" @click="refreshTab">
                <el-tooltip effect="dark" content="刷新" placement="bottom" >
                     <i class="el-icon-refresh"></i>
                </el-tooltip>
            </div>
        </el-col>
        <el-col :xs="14" :sm="7" :md="6" :lg="5" :xl="5">
            <!-- 用户头像 -->
            <el-col :xs="16" :sm="16" :md="16" :lg="12" :xl="12">
                <div class="user-avator" style="float: left">
                    <!--<img src="../../assets/img/img.jpg" />-->
                </div>
                <div style="float: left">
                    <!-- 用户名下拉菜单 -->
                    <el-dropdown class="user-name" trigger="click" @command="handleCommand" style="float: left" size="mini" >
                        <span class="el-dropdown-link">
                            {{username}}
                            <i class="el-icon-caret-bottom"></i>
                        </span>
                        <el-dropdown-menu slot="dropdown">
                            <a href="https://github.com/zhangjiangmse/back_menu" target="_blank">
                                <el-dropdown-item>项目仓库</el-dropdown-item>
                            </a>
                            <el-dropdown-item divided command="loginout">退出登录</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </div>
            </el-col>

<!--            <el-col :xs="8" :sm="8" :md="8" :lg="12" :xl="12">-->
<!--                &lt;!&ndash; 语言下拉菜单 &ndash;&gt;-->
<!--                <el-dropdown class="user-name" trigger="click" @command="handleCommand_lang" style="float: left" size="mini">-->
<!--                        <span class="el-dropdown-link">-->
<!--                           {{currentLanguage}}-->
<!--                            <i class="el-icon-caret-bottom"></i>-->
<!--                        </span>-->
<!--                    <el-dropdown-menu slot="dropdown">-->
<!--                        <el-dropdown-item command="cn">{{$t('message.cn')}}</el-dropdown-item>-->
<!--                        <el-dropdown-item divided command="en">{{$t('message.en')}}</el-dropdown-item>-->
<!--                    </el-dropdown-menu>-->
<!--                </el-dropdown>-->
<!--            </el-col>-->
        </el-col>
    </el-row>
</template>
<script>
import bus from '../common/bus';
export default {
    inject: ['reload'],
    data() {
        return {
            currentLanguage:'中文',
            collapse: false,
            fullscreen: false,
            name: 'admin',
            message: 2
        };
    },
    computed: {
        username() {
            let username = localStorage.getItem('ms_username');
            return username ? username : this.name;
        }
    },

    methods: {
        // 用户名下拉菜单选择事件
        handleCommand(command) {
            if (command == 'loginout') {
                localStorage.removeItem('ms_username');
                this.$router.push('/login');
            }
        },
        // 语言切换下拉菜单选择事件
        handleCommand_lang(command) {
            this.currentLanguage = this.$t('message[\''+command+'\']')
            this.$i18n.locale = command
            this.$notify({
                title: this.$t('message.tip'),
                message: this.$t('message.Switch_language_successfully'),
                position: 'bottom-right'
            });
        },
        // 侧边栏折叠
        collapseChage() {

            this.collapse = !this.collapse;
            bus.$emit('collapse', this.collapse);
        },
        //刷新Tab
        refreshTab(){
            this.reload()
        },
        // 全屏事件
        handleFullScreen() {
            let element = document.documentElement;
            if (this.fullscreen) {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                } else if (document.webkitCancelFullScreen) {
                    document.webkitCancelFullScreen();
                } else if (document.mozCancelFullScreen) {
                    document.mozCancelFullScreen();
                } else if (document.msExitFullscreen) {
                    document.msExitFullscreen();
                }
            } else {
                if (element.requestFullscreen) {
                    element.requestFullscreen();
                } else if (element.webkitRequestFullScreen) {
                    element.webkitRequestFullScreen();
                } else if (element.mozRequestFullScreen) {
                    element.mozRequestFullScreen();
                } else if (element.msRequestFullscreen) {
                    // IE11
                    element.msRequestFullscreen();
                }
            }
            this.fullscreen = !this.fullscreen;
        }
    },
    mounted() {
        if (document.body.clientWidth < 1500) {
            this.collapseChage();
        }
    }
};
</script>
<style scoped>

.header {
    position: relative;
    box-sizing: border-box;
    width: 100%;
    height: 70px;
    font-size: 22px;
    color: #fff;
}
div .logo {
    color: #ffffff;
    font-size: 20px;
    text-align: center;
    line-height: 60px;
}

.header-right {
    float: right;
    padding-right: 50px;
}
.header-user-con {
    display: flex;
    height: 70px;
    align-items: center;
}
.btn-refresh {
    float: right;
    line-height: 67px;
    color: white;
    font-size: 23px;
    margin-left: 7px;
    margin-right: 7px;
    cursor: pointer;
}
.btn-fullscreen {
    color: white;
    transform: rotate(45deg);
    margin-left: 40px;
    font-size: 24px;
    width: 0px;
    height: 0px;
    line-height: 70px;
    cursor: pointer;
}
.btn-bell {
    float: right;
    height: 50px;
    margin-left: 3px;
    margin-right: 20px;
    border-radius: 15px;
    cursor: pointer;
    line-height: 70px;
}
.btn-bell-badge {
    position: absolute;
    top: 18px;
    width: 8px;
    height: 8px;
    border-radius: 4px;
    background: #f56c6c;
    color: #fff;

}
.btn-bell .el-icon-bell {
    font-size: 23px;
    color: #fff;
}
.user-name {
    margin-top: 20px;
}
.user-avator {
    width: 45px;
    margin-top: 10px;
}
.user-avator img {
    display: block;
    width: 40px;
    height: 40px;
    border-radius: 50%;
}
.el-dropdown-link {
    color: #fff;
    cursor: pointer;
}
.el-dropdown-menu__item {
    text-align: center;
}
</style>
