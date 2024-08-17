<template>
    <router-link to="/articleDetail">
        <div class="level2">
            <div class="news">
                <div class="article">
                    Tomato price market: In the next 5 years, if you want to make money from tomatoes, you can't do without these points
                </div>
                <div class="articleImgs">
                    <img class="news-img" src="../assets/icons/tomato.svg" alt="Description of the image">
                </div>
            </div>
        </div>
    </router-link>
</template>

<script>
export default {
    props: {

    },
    setup(props) {

    }
}
</script>

<style scoped>
.level2 {
    display: flex;
    justify-content: center;
    margin: 0 10px 10px;
}

.news {
    display: flex;
    justify-content: space-between; /* 两端对齐 */
    align-items: center;
    width: 100%; /* 确保容器占满父元素 */
}

.article {
    flex: 1; /* 占据剩余空间 */
}

.news-img {
    width: 120px;
    height: 110px;
    margin-left: 10px;
}
</style>
