<template>
  <div>
    <div class="list-container">
      <div v-for="(topic, topicIndex) in paginatedTopics" :key="topic.guid" class="topic-card">
        <div class="title-container">
          <div v-if="!isEditing(topicIndex)">
            <h3>Topic: {{ topic.name }}</h3>
            <div v-if="!isAddingComment(topicIndex, topic.guid)" class="edit-btn">
              <button @click="startEditing(topicIndex)">Edit Topic</button>
              <button @click="startAddingComment(topicIndex)">+ Add Comment</button>
            </div>
          </div>
          <div v-else>
            <label for="topic">Topic:</label>
            <input type="text" v-model="editTopic" name="topic">
            <div class="edit-btn">
              <button @click="saveChanges(topicIndex)">Save</button>
              <button @click="cancelEdit">Cancel</button>
            </div>
          </div>
        </div>

        <!-- Template for adding a new comment -->
        <div v-if="isAddingComment(topicIndex, topic.guid)">
          <hr>
          <h4>Adding a new comment</h4>
          <div class="form-fields">
            <label for="newComment">Comment:</label>
            <textarea type="text" v-model="newComment" name="newComment" rows="4" cols="50"></textarea>
          </div>
          <div class="form-fields">
            <label for="by">Author:</label>
            <input type="text" v-model="newCommentAuthor" name="by">
          </div>
          <div class="edit-btn">
            <button @click="saveNewComment(topicIndex)">+ Save Comment</button>
            <button @click="cancelEdit">Cancel</button>
          </div>
        </div>

          <!-- Template for listing out the topics -->
        <div v-for="(comment, commentIndex) in topic.comments" :key="commentIndex" class="comment-container">
          <div v-if="!isEditing(topicIndex, commentIndex)">
            <p><strong>Comment: </strong>{{ comment.comment }}</p>
            <small>By: {{ comment.by }} on {{ new Date(comment.date).toLocaleDateString() }}</small>
            <div class="edit-btn">
              <button @click="startEditing(topicIndex, commentIndex)">Edit</button>
              <button @click="deleteComment(topicIndex, commentIndex)">Delete</button>
            </div>
          </div>
          <div v-else>
            <div>
              <label for="comment">Comment:</label>
              <textarea  type="text" v-model="editContent" name="comment" rows="4" cols="50"></textarea>
            </div>
            <small>By: {{ comment.by }} on {{ new Date(comment.date).toLocaleDateString() }}</small>
            <div class="edit-btn">
              <button @click="saveChanges(topicIndex, commentIndex)">Save</button>
              <button @click="cancelEdit">Cancel</button>
            </div>
          </div>
        </div>

        
        <div v-if="topic.comments.length == 0">
          <br>
          <hr>
          <p>No comments for this topic</p>
        </div>
        <hr>
      </div>
    </div>
    <Pagination
      :currentPage="currentPage"
      :totalPages="totalPages"
      @page-changed="handlePageChange"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import Pagination from './Pagination.vue';
import topicsData from '../assets/topics.json';

const topics = ref([]);
const editingComment = ref({ topicIndex: null, commentIndex: null });
const editingTopic = ref({ topicIndex: null });
const addingComment = ref({ topicIndex: null });
const editContent = ref('');
const editTopic = ref('');
const newComment = ref('');
const newCommentAuthor = ref('');

onMounted(() => {
  const getLocalStorageTopics = JSON.parse(localStorage.getItem('topics'));
  if (getLocalStorageTopics) {
    topics.value = getLocalStorageTopics
  } else {
    topics.value = topicsData.topics;
  }
});

const currentPage = ref(1);
const itemsPerPage = ref(20);

const totalPages = computed(() => {
  return Math.ceil(topics.value.length / itemsPerPage.value);
});

const paginatedTopics = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return topics.value.slice(start, end);
});

const isEditing = (topicIndex, commentIndex) => {
  if (commentIndex == undefined) {
    return editingTopic.value.topicIndex === topicIndex
  } else {
    return editingComment.value.topicIndex === topicIndex && editingComment.value.commentIndex === commentIndex;
  }
};

const startEditing = (topicIndex, commentIndex) => {
  if (commentIndex === undefined) {
    editingTopic.value = { topicIndex };
    editTopic.value = topics.value[topicIndex].name
  } else {
    editingComment.value = { topicIndex, commentIndex };
    editContent.value = topics.value[topicIndex].comments[commentIndex].comment;
  }
};

const saveChanges = (topicIndex, commentIndex) => {
  if (commentIndex === undefined) {
    topics.value[topicIndex].name = editTopic.value;
    editingTopic.value = { topicIndex: null };
  } else {
    topics.value[topicIndex].comments[commentIndex].comment = editContent.value;
    editingComment.value = { topicIndex: null, commentIndex: null };
  }
  editContent.value = '';
  editTopic.value = '';
  saveToLocalStorage();
};

const cancelEdit = () => {
  editingTopic.value = { topicIndex: null };
  editingComment.value = { topicIndex: null, commentIndex: null };
  addingComment.value = { topicIndex: null };
  editContent.value = '';
  editTopic.value = '';
  newComment.value = '';
  newCommentAuthor.value = '';
};

const isAddingComment = (topicIndex, guid) => {
  return addingComment.value.topicIndex === topicIndex
}

const startAddingComment = (topicIndex) => {
  addingComment.value = { topicIndex };
};

const saveNewComment = (topicIndex) => {
  editContent.value = topics.value[topicIndex].comments.push({
      comment: newComment.value,
      by: newCommentAuthor.value,
      date: new Date()
    })
    addingComment.value = { topicIndex: null };
    newComment.value = '';
    newCommentAuthor.value = '';
    saveToLocalStorage()
}

const handlePageChange = (page) => {
  console.log(page);
  currentPage.value = page;
};

const saveToLocalStorage = () => {
  localStorage.setItem('topics',JSON.stringify(topics.value))
}

const deleteComment = (topicIndex, commentIndex) => {
  topics.value[topicIndex].comments.splice(commentIndex, 1)
  saveToLocalStorage();
}

</script>

<style scoped>

.list-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px; /* Adjust the gap between cards */
}

.topic-card {
  background: rgb(39, 39, 103);
  padding: 16px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.comment-container {
  border-top: 1px solid #ccc;
  padding: 8px 0;
  margin: 8px 0;
}

.edit-btn {
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
}

.form-fields, label {
  margin-right: 15px;
}

.form-fields {
  display: flex;
  justify-content: space-between;
}
</style>