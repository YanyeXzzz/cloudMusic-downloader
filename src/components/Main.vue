<template>
    <div class="box">
        <h1>MusicDownloader</h1>
        <div style="display: flex;">
            <el-input placeholder="请输入要搜索的歌曲" style="margin-right: 10px;" v-model="songName" @change="search"></el-input>
            <el-button type="primary" @click="search">搜索</el-button>
        </div>
        <el-table :data="tableData" stripe style="width: 100%">
            <el-table-column prop="songName" label="歌曲名" width="180" />
            <el-table-column prop="singer" label="歌手" width="180" />
            <el-table-column prop="time" label="时长" />
            <el-table-column prop="action" label="操作">
                <template #default="scope">
                    <el-button @click="play(scope.row)">播放</el-button>
                    <el-button type="primary" @click="save(scope.$index)">下载</el-button>
                </template>
            </el-table-column>

        </el-table>
        <!-- <footer>Created By Yanye</footer> -->
        <el-card class="card" shadow="hover">
            <p>Github:<el-button link type="primary">https://github.com/YanyeXzzz/cloudMusic-downloader.git</el-button>
            </p>
            Created By Yanye
        </el-card>
    </div>
    <div class="frame" ref="frame" @mouseenter="hoverFrame">
        <iframe :src="iframUrl"
            frameborder="0" ></iframe>
    </div>


</template>
<script setup>
import { ref, onBeforeMount, onMounted } from 'vue'
import axios from 'axios'
import { ElMessage } from 'element-plus';
import { ElMessageBox } from 'element-plus'

onBeforeMount(() => {
    // ElMessageBox.alert('该工具仅用于学习！', '注意', {
    //     confirmButtonText: 'OK',
    //     type: 'warning'
    // })
})

onMounted(() => {
    setInterval(() => {
        frame.value.children[0].style.opacity = '0'
    },10000)
})

const songName = ref()
const songs = []
const frame = ref()
const iframUrl = ref('')


const hoverFrame = () => {
    frame.value.children[0].style.opacity = '1'
}

const play = (row) => {
    iframUrl.value = 'https://music.163.com/outchain/player?type=2&id=' + row.songId + '&auto=1&height=66&bg=e8e8e8'
    frame.value.children[0].style.opacity = '1'

}

const search = () => {
    tableData.value = []
    if (songName.value == '' || songName.value == undefined) {
        ElMessage({
            message: '歌曲名不能为空!',
            type: 'warning'
        })
        return
    }
    axios.get('http://localhost:8080/search?songName=' + songName.value + '&limit=' + 20)
        .then(response => {
            songs.value = response.data.data.result.songs
            for (let i = 0; i < songs.value.length; i++) {
                const songInfo = {
                    songName: songs.value[i].name,
                    singer: songs.value[i].artists[0].name,
                    time: '',
                    songId: songs.value[i].id
                }
                tableData.value.push(songInfo)
            }
        })
        .catch(err => {
            console.log(err)
        })
}

const save = (index) => {
    // http://music.163.com/song/media/outer/url?id=
    const url = 'http://music.163.com/song/media/outer/url?id=' + songs.value[index].id
    axios.get('http://localhost:8080/save?url=' + url + '&songName=' + songs.value[index].name)
        .then(response => {
            const downloadUrl = response.data.data
            const a = document.createElement('a')
            a.href = downloadUrl
            a.download = ''
            a.click()
            a.remove()
        })
        .catch(err => {
            console.log(err)
        })
}


const tableData = ref([

])
</script>
<style>
.box {
    width: 50%;
    height: 650px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    align-items: center;
}

footer {
    width: 100%;
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f4f4f4;
}

.card {
    width: 50%;
    text-align: center;
    position: fixed;
    left: 50%;
    top: 80%;
    transform: translateX(-50%);
}

.frame {
    width: 100%;
    height: 10vh;
    position: absolute;
    left: 0;
    bottom: 0;
    
}

iframe {
    width: 100%;
    height: 100%;
    transition: all 1s;
}
</style>
