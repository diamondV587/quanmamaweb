<!-- hello -->
<template>
    <div class='scroll-wrapper'>
        <div ref="scroll" class="wrap">
            <div class="scroll-area">
                <!-- <div v-for="n in 50" v-bind:key="n" class="item">{{ n }}</div> -->
                <div :class="{hide: status===0}" class="refresh">{{ txt }}</div>
                <div ref="sticky" class="sticky" :top="0">我是分割线</div>
                <div v-for="n in 50" v-bind:key="n+'r'" class="item">{{ n+20 }}</div>
                <div ref="sticky2" class="sticky" :top="40">我是分割线2</div>
                <div v-for="n in 50" v-bind:key="n+'t'" class="item">{{ n+70 }}</div>
            </div>
        </div>
    </div>
</template>

<script>
import IScroll from 'iscroll/build/iscroll-probe';
import enableSticky from '../../../static/js/iscroll-sticky';

enableSticky(IScroll);

function getData(){
    return new Promise((resolve, reject) =>{
        var timeOut = Math.random() * 2;
        console.log('set timeout to: ' + timeOut + ' seconds.');
        setTimeout(function () {
            resolve();
        }, timeOut * 1000);
    });
}

export default {
    name:"ScrollWrapper",
    data() {
        return {
            scroll:null,
            status: 0,
            txt:"下拉刷新"
        }
    },
    watch: {
        status() {
            this.scroll.refresh();
        }
    },
    mounted() {
        const el = this.$refs.scroll;

        this.scroll = new IScroll('.wrap',{
            snap:true,
            // snap:'.item',
            // scrollbars:true,
            // shrinkScrollbars:'scale',
            mouseWheel:true,
            probeType:3
        })

        const stickyEl = this.$refs.sticky;
        const stickyEl2 = this.$refs.sticky2;

        // 允许元素对象集合sticky
        this.scroll.enableStickyHeaders([
            {
            el: stickyEl,
            top: stickyEl.getAttribute('top') // 此处我把top值配置在了原生prop
            },
            {
            el: stickyEl2,
            top: stickyEl2.getAttribute('top') // 此处我把top值配置在了原生prop
            }
        ]);


        this.scroll.on('scroll',e=>{
            console.log(this.scroll.y);

            const y = this.scroll.y;
            if(y >= 50){
                this.txt = "释放刷新";
                this.status = 1;
            } else if (y > 0){
                this.txt = "下拉刷新";
                this.status = 0;
            }
        })

        
        // this.scroll.on('scroll', e => {
        // // 此处scrollEl是容器高度，contentEl是内容高度，因为y是负值，所以用scrollEl - contentEl
        // if (this.scroll.y <= scrollEl.offsetHeight - contentEl.offsetHeight) {
        //     // do something 上拉加载
        // }
        // });

        this.scroll.on("scrollEnd",e=>{
            if(status === 1) {
                this.txt = "刷新中。。。";
                this.status = 2;
                this.scroll.disable();
                this.updateData();
            }
        });

        // getData().then(()=>{
        //     console.log("gedata-refresh")
        //     this.scroll.refresh();
        // })

        // el.addEventListener('touchstart',()=>{
        //     console.log('rerefsh')
        //     this.scroll.refresh();
        // });

        // this.scroll.goToPage(0,30,1000);
        // this.scroll.prev();
        //this.scroll.next();
    },
    methods:{
        updateData(){
            console.log('update data');
            getData().then(_=>{
                // 数据更新完成
                this.txt = '刷新完成';
                // 延迟1秒后继续隐藏刷新文本
                setTimeout(_=>{
                this.txt = '下拉刷新';
                this.status = 0; // 状态重置为0
                this.scroll.enable();
                }, 1000);
            })
        }
    }
};

</script>

<style scoped>
    .scroll-wrapper {
        width: 100%;
    }
    .wrap {
        height: 400px;
        overflow: hidden;
    }
    .sticky {
        width: 100%;
        height: 20px;
        text-align: center;
        line-height: 20px;
        color: #ffffff;
        background: blue;
    }
    .refresh{
        width: 100%;
        height: 50px;
        line-height: 50px;
        text-align: center;
    }
    .scroll-area .hide{
        /* 当status为0默认时，隐藏刷新文本，通过定位到容器外面 */
        position: absolute;
        left: 0;
        top: -50px;
    }
</style>