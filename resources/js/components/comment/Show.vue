<template>
    <main>
        <div class="container mt-5">
            <div class="d-flex justify-content-center row">
                    <li class="d-flex flex-column comment-section comment" v-for="comment in comments" :key="comment.id">
                        <div class="p-2">
                            <div class="d-flex flex-row user-info">
                                <div class="d-flex flex-column justify-content-start ml-2">
                                    <span class="d-block font-weight-bold name">{{ comment.principal.us_name }}</span>
                                    <span class="fs-12 text-black-50">Shared - Jan 2020</span>
                                </div>
                            </div>
                            <div class="mt-2">
                                <p class="fs-11">{{ comment.principal.content }}</p>
                            </div>
                            <button class="comment__button" @click="chatVisible(comment.principal.id)">Reply</button>

                            <div v-if="isVisible == 0"></div>
                            <div :id="comment.principal.id" v-else-if="isVisible == comment.principal.id">
                                <form v-if="chatVisible" @submit.prevent="replyComment">
                                    <div class="bg-light p-2">
                                        <div class="d-flex flex-row align-items-start">
                                                <div class="card-body">
                                                    <form @submit.prevent="replyComment">

                                                        <div class="form-floating mb-3">
                                                            <input type="text" class="form-control" id="floatingInput" v-model="replay_us_name">
                                                            <label for="floatingInput">User Name</label>
                                                        </div>
                                                        <div class="form-floating">
                                                            <input type="text" class="form-control" id="floatingPassword" v-model="replay_content">
                                                            <label for="floatingPassword">Comment</label>
                                                        </div>
                                                    </form>
                                                </div>

                                        </div>
                                        <div class="mt-2 text-right">
                                            <button class="btn btn-primary btn-sm shadow-none" type="submit">Post replay</button>
                                            <button class="btn btn-outline-primary btn-sm ml-1 shadow-none" @click="isVisible = 0" type="button">Cancel</button>
                                        </div>
                                    </div>
                                </form>
                            </div>

                        </div>

                        <!-- Comments in comments -->
                        <ul class="comment" v-for="subcomments in comment.subcomments" :key="subcomments.id">
                           <div class="d-flex flex-row user-info">
                                <div class="d-flex flex-column justify-content-start ml-2">
                                    <span class="d-block font-weight-bold name">{{ subcomments.us_name }}</span>
                                    <span class="fs-12 text-black-50">Shared - Jan 2020</span>
                                </div>
                            </div>
                            <div class="mt-2">
                                <p class="fs-11">{{ subcomments.content }}</p>
                            </div>
                        </ul>
                        <!-- Comments in comments -->
                    </li>


                    <form @submit.prevent="create">
                        <div class="bg-light p-2">
                            <div class="d-flex flex-row align-items-start">
                                <img class="rounded-circle" src="https://i.imgur.com/RpzrMR2.jpg" width="40">
                                    <div class="card-body">
                                        <form @submit.prevent="create">

                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" id="floatingInput" v-model="comment.us_name">
                                                <label for="floatingInput">User Name</label>
                                            </div>
                                            <div class="form-floating">
                                                <input type="text" class="form-control" id="floatingPassword" v-model="comment.content">
                                                <label for="floatingPassword">Comment</label>
                                            </div>
                                        </form>
                                    </div>

                            </div>
                            <div class="mt-2 text-right">
                                <button class="btn btn-primary btn-sm shadow-none" type="submit">Post comment</button>
                                <button class="btn btn-outline-primary btn-sm ml-1 shadow-none" type="button">Cancel</button>
                            </div>
                        </div>
                    </form>

            </div>
        </div>

    </main>
</template>

<script>
export default {
    data() {
        return {
            id: null,
            isVisible: 0,
            comments: [],
            replay_us_name: '',
            replay_content: '',
            comment: {
                us_name: "",
                content: "",
                sub_comment_id: ""
            }

        }
    },
    mounted() {
        this.id = this._uid,
        this.axios.get('/api/comment')
            .then(response => {
                this.comments = response.data
            })
            .catch(error => {
                console.log(error)
            })
    },
    methods: {
        async create() {
            this.comment.sub_comment_id = ""
            await this.axios.post('/api/comment', this.comment)
                .then(response => {
                    this.comments.unshift({principal: response.data.blog})
                    this.cancel()
                })
                .catch(error => {
                    console.log(error)
                })
        },
        async replyComment() {

            await this.axios({
                method: 'post',
                url: '/api/comment',
                data: {
                    us_name: this.replay_us_name,
                    content: this.replay_content,
                    sub_comment_id: this.comment.sub_comment_id
                }
            }).then(response => {
                var arr1 = this.comments.filter(d => d.principal.id == this.comment.sub_comment_id)
                var pushData = arr1[0].subcomments

                pushData.unshift(response.data.blog)

                this.replay_content = ''
                this.replay_us_name = ''


            }).catch(error => {
                console.log(error)
            })

        },
        chatVisible(id) {
            this.comment.sub_comment_id = id,
            this.isVisible = id
        },
        cancel() {
            this.comment.content = ''
            this.comment.us_name = ''
        }
    }
}
</script>

<style >

.shadow-none {
    box-shadow: none
}

.name {
    color: #007bff
}

.cursor:hover {
    color: blue
}

.cursor {
    cursor: pointer
}

.textarea {
    resize: none
}

.comment__button {
  display: block;
  color: #1976d2;
  font-size: 12px;
  margin: 5px 0;
  padding: 0;
  border: 0;
  cursor: pointer;


}

.comment {
  margin: 20px 0;
  padding-left: 10px;
  list-style: none;
  border-left: 2px solid #eee;
}
</style>
