<template>
  <v-container>
    <!-- 显示加载指示器，水平垂直居中 -->
    <v-row v-if="isLoading" justify="center" align="center" style="min-height: 80vh;">
      <v-col cols="12" class="text-center">
        <v-progress-circular indeterminate color="primary" size="70" width="7"></v-progress-circular>
      </v-col>
    </v-row>

    <!-- 加载完成后显示文章列表 -->
    <v-row v-else>
      <v-col class="text-left">
        <v-btn color="#E1F5FE" @click="addArticle">
          <v-icon left>mdi-plus</v-icon> 添加文章
        </v-btn>
      </v-col>
    </v-row>

    <v-table v-if="!isLoading">
      <thead>
        <tr>
          <th>标题</th>
          <th>文章详情</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(article, index) in articles" :key="index">
          <td class="article-title">{{ article.title }}</td>
          <td class="article-detail"> 
            {{ article.content }}
          </td>
          <td>
            <v-btn color="#E1F5FE" small class="mx-2" @click="editArticle(article)">
              <v-icon left>mdi-pencil</v-icon> 编辑
            </v-btn>
            <v-btn color="#FFCCBC" small class="mx-2" @click="deleteArticle(article)">
              <v-icon left>mdi-delete</v-icon> 删除
            </v-btn>
          </td>
        </tr>
      </tbody>
    </v-table>

    <!-- 添加/编辑文章模态框 -->
    <v-dialog v-model="dialogVisible" max-width="600px">
      <v-card>
        <v-card-title>{{ editMode ? '编辑文章' : '添加文章' }}</v-card-title>
        <v-card-text>
          <v-text-field label="文章标题" v-model="currentArticle.title"></v-text-field>
          <v-textarea label="文章内容" v-model="currentArticle.content"></v-textarea>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="closeDialog">
            <v-icon left>mdi-cancel</v-icon> 取消
          </v-btn>
          <v-btn color="primary" @click="saveArticle">
            <v-icon left>mdi-content-save</v-icon> 保存
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Snackbar for notifications -->
    <v-snackbar v-model="snackbar.show" :color="snackbar.color" top right>
      {{ snackbar.text }}
    </v-snackbar>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      articles: [], // 存储文章列表
      showAddDialog: false,
      showEditDialog: false,
      currentArticle: { id: null, title: "", content: "" },
      editMode: false,
      isLoading: true, // 加载状态
      snackbar: {
        show: false, // 控制 snackbar 显示
        text: '', // 消息文本
        color: 'success', // snackbar 颜色 (success, error 等)
      },
    };
  },
  computed: {
    dialogVisible() {
      return this.showAddDialog || this.showEditDialog;
    },
  },
  created() {
    this.fetchArticles();
  },
  methods: {
    // 获取所有文章
    async fetchArticles() {
      this.isLoading = true; // 开始加载
      try {
        const response = await fetch('/api/articles');
        const data = await response.json();
        this.articles = data.articles;
      } catch (error) {
        this.showMessage('获取文章失败', 'error');
        console.error('Failed to fetch articles:', error);
      } finally {
        this.isLoading = false; // 加载完成
      }
    },
    // 添加文章
    addArticle() {
      this.currentArticle = { id: null, title: "", content: "" }; // 重置文章数据
      this.editMode = false;
      this.showAddDialog = true;
    },
    // 编辑文章
    editArticle(article) {
      this.currentArticle = { ...article }; // 克隆文章对象，避免直接修改
      this.editMode = true;
      this.showEditDialog = true;
    },
    // 保存文章（新增或编辑）
    async saveArticle() {
      try {
        if (this.editMode) {
          // 更新文章
          const response = await fetch(`/api/articles/${this.currentArticle.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(this.currentArticle),
          });
          if (response.ok) {
            const index = this.articles.findIndex((a) => a.id === this.currentArticle.id);
            if (index !== -1) {
              this.articles.splice(index, 1, this.currentArticle); // 更新文章列表中的文章
            }
            this.showMessage('文章更新成功', 'success');
          } else {
            this.showMessage('文章更新失败', 'error');
          }
        } else {
          // 添加新文章
          const response = await fetch('/api/articles', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(this.currentArticle),
          });
          if (response.ok) {
            this.articles.unshift(this.currentArticle); // 将新文章添加到第一位
            this.showMessage('文章添加成功', 'success');
          } else {
            this.showMessage('文章添加失败', 'error');
          }
        }
      } catch (error) {
        this.showMessage('保存文章时出错', 'error');
        console.error('Error saving article:', error);
      } finally {
        this.closeDialog();
      }
    },
    // 删除文章
    async deleteArticle(article) {
      try {
        const response = await fetch(`/api/articles/${article.id}`, { method: 'DELETE' });
        if (response.ok) {
          this.articles = this.articles.filter((a) => a.id !== article.id); // 动态移除文章
          this.showMessage('文章删除成功', 'success');
        } else {
          this.showMessage('文章删除失败', 'error');
        }
      } catch (error) {
        this.showMessage('删除文章时出错', 'error');
        console.error('Error deleting article:', error);
      }
    },
    // 关闭模态框
    closeDialog() {
      this.showAddDialog = false;
      this.showEditDialog = false;
    },
    // 显示消息的函数
    showMessage(message, color) {
      this.snackbar.text = message;
      this.snackbar.color = color;
      this.snackbar.show = true;
    },
  },
};
</script>

<style scoped>
.fill-height {
  height: 100vh;
}

.v-card {
  background-color: white;
  border-radius: 15px;
}
.article-title {
  max-width: 130px; /* 增加宽度 */

}
.article-detail {
  color: gray;
  max-width: 350px; /* 增加宽度 */
  font-size: 0.85em; /* 调整字体大小 */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
