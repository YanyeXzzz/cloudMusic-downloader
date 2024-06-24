<template>
    <div class="box">
        <h1>MusicDownloader</h1>
        <div style="display: flex;">
            <el-input placeholder="请输入要搜索的歌曲" style="margin-right: 10px;" v-model="songName"></el-input>
            <el-button type="primary" @click="search">搜索</el-button>
        </div>
        <el-table :data="tableData" stripe style="width: 100%">
            <el-table-column prop="songName" label="歌曲名" width="180" />
            <el-table-column prop="singer" label="歌手" width="180" />
            <el-table-column prop="time" label="时长" />
            <el-table-column prop="action" label="操作">
                <template #default="scope">
                    <el-button type="primary" @click="save(scope.$index)">下载</el-button>
                </template>
            </el-table-column>
            
        </el-table>
        <!-- <footer>Created By Yanye</footer> -->
        <el-card class="card" shadow="hover">
            <p>Github:<el-button link type="primary">https://github.com/YanyeXzzz/cloudMusic-downloader.git</el-button></p>
            
            <p>Created By Yanye</p>
            
        </el-card>
    </div>

</template>
<script setup>
    import { ref, onMounted} from 'vue'
    import axios from 'axios'
    import { ElMessage } from 'element-plus';
    import { ElMessageBox } from 'element-plus'

    onMounted(() => {
        // ElMessageBox.alert('该工具仅用于学习，请勿用于商业用途！', '注意', {
        //     confirmButtonText: 'OK',
        //     type: 'warning'
        // })
    })

    const songName = ref()
    const songs = []

    const search = () => {
        tableData.value = []
        if (songName.value == '' || songName.value == undefined){
            ElMessage({
                message: '歌曲名不能为空!',
                type: 'warning'
            })
            return
        }
        axios.get('http://47.109.25.111:8080/search?songName=' + songName.value + '&limit=' + 20)
        .then(response => {
            songs.value = response.data.data.result.songs
            for (let i = 0; i < songs.value.length; i++){
                const songInfo = {
                    songName: songs.value[i].name,
                    singer: songs.value[i].artists[0].name,
                    time: ''
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
        axios.get('http://47.109.25.111:8080/save?url=' + url + '&songName=' + songs.value[index].name)
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

footer{
    width: 100%;
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f4f4f4;
}

.card{
    width: 50%;
    text-align: center;
    position: fixed;
    left: 50%;
    top: 70%;
    transform: translateX(-50%);
}
</style>
