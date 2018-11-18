<style lang="scss" scoped>
    .magnify {
        position: relative;
        margin: 0 20px;

        .preview-box {
            position: relative;    
            width: 480px;
            height: 270px;
            border: 1px solid darkgray;
            // margin: 0 auto;

            &:hover .hover-box {
                display: block;
            }
            .hover-box {
                position: absolute;
                display: none;
                left: 0;
                top: 0;
                width: 100px;
                height: 100px;
                border: 1px solid deepskyblue;
                background: deepskyblue;
                opacity: 0.2;
                cursor: move;
                user-select: none;
            }
        }
    }
    .zoom-box {
        position: relative;
        width: 480px;
        height: 450px;
        overflow: hidden;
        // left: 0;
        // top: 0;
        img {
            position: absolute;
            top: 0;
            left: 0;
        }
    }
</style>
<template>
    <div class="magnify">
        <el-popover
            placement="right-start"
            width="480"
            trigger="hover">
            <div class="zoom-box"  ref="zoomBox">
                <img :src="zoomImg" alt="" ref="bigImg">
            </div>
            <div class="preview-box" ref="previewBox" slot="reference" @mousemove="move($event)"  @mouseout="out">
                <img width="100%" :src="previewImg" alt="">
                <div class="hover-box" ref="hoverBox"></div>
            </div>

        </el-popover>
   </div>
</template>
<script>
export default {
    props: {
        previewImg: {
            type: String,
            default: ''
        },
        zoomImg: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            zoomVisiable: true,
            hoverVisiable: false
        };
    },
    methods: {
        init() {
            this.oHoverBox = this.$refs.hoverBox; // 鼠标预览框
            this.oPreviewBox = this.$refs.previewBox; // 预览图盒子
            this.oBigImg = this.$refs.bigImg; //放大图盒子
            this.imgBox = this.$refs.zoomBox; // 放大图

            // 鼠标hover效果
            this.houverWidth = this.oHoverBox.offsetWidth;
            this.houverHeight = this.oHoverBox.offsetHeight;
            // 预览图盒子
            this.pWidth = this.oPreviewBox.offsetWidth;
            this.pHeight = this.oPreviewBox.offsetHeight;
            // 图片放大框
            this.imgWidth = this.oBigImg.offsetWidth;
            this.imgHeight = this.oBigImg.offsetHeight;
            // 放大图片的实体
            this.bWidth = this.imgBox.offsetWidth;
            this.bHeight = this.imgBox.offsetHeight;
            // 鼠标滚动
            this.scroll = document.documentElement.scrollTop || document.body.scrollTop;
        },
        out() {
            this.zoomVisiable = false;
        },
        offset(el) {
            let top = el.offsetTop;
            let left = el.offsetLeft;

            while (el.offsetParent) {
                el = el.offsetParent;
                top += el.offsetTop;
                left += el.offsetLeft;
            }
            return {
                left: left,
                top: top
            }
        },
        move(ev) {
            // 初始化参数
            this.init();
            // 鼠标距离屏幕位置
            let moveX = ev.clientX,
                moveY = ev.clientY;

            console.log(moveX, moveY);

            // 大盒子距离顶部位置
            let offsetLeft = this.offset(this.oPreviewBox).left,
                offsetTop = this.offset(this.oPreviewBox).top;

           // console.log(offsetLeft, offsetTop);

            // 计算左边位置
            let left = moveX - offsetLeft - this.houverWidth / 2;
            let top;

            // 鼠标滚动的顶部
            if (this.scroll > 0) {
                top = moveY - offsetTop + this.scroll - this.houverHeight / 2;
            } else {
                top = moveY - offsetTop - this.houverHeight / 2;
            }
            // 最大的宽度（预览图的宽减去鼠标hover的宽）;最大高度 (预览图的高减去鼠标hover的高)
            let maxWidth = this.pWidth - this.houverWidth,
                maxHeight = this.pHeight - this.houverHeight;

            // console.log('maxWidth:', maxWidth, 'maxHeight:', maxHeight);


            left = (left < 0) ? 0 : ((left > maxWidth) ? maxWidth : left);
            top = (top < 0 )? 0 : ((top > maxHeight) ? maxHeight : top);
            // console.log('left:', left, 'top:', top);
            

            let percentX = left / (maxWidth),
                percentY = top / (maxHeight);

            // 动态更改鼠标hover位置
            this.oHoverBox.style.left = left + 'px';
            this.oHoverBox.style.top = top  + 'px';

            // 动态更改放大图的位置
            this.oBigImg.style.left = percentX * (this.bWidth - this.imgWidth) + 'px';
            this.oBigImg.style.top = percentY * (this.bHeight - this.imgHeight) + 'px';

            // this.$emit('move', ev); // 分发事件
            this.zoomVisiable = true; // 放开放大图
        }
    }
}
</script>
