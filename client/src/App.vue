<template>
  <div>
    <div class="todo-container">
      <h2>Quản lý công việc</h2>

      <!-- Hiển thị hiệu ứng loading khi dữ liệu đang tải -->
      <div v-if="isLoading" class="loading">Đang tải dữ liệu...</div>

      <!-- Ẩn hiệu ứng loading và hiển thị nội dung khi dữ liệu đã tải -->
      <div v-else>
        <div class="input-container">
          <input v-model="inputName" placeholder="Nhập tên công việc" />

          <button @click="handleAdd" class="add-btn">Thêm công việc</button>
        </div>
        <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
        <div class="filter-container">
          <button>Tất cả</button>
          <button>Hoàn thành</button>
          <button>Đang thực hiện</button>
        </div>
        <ul id="work-list">
          <li
            v-for="work in works"
            :key="work.id"
            :class="{ completed: work.status }"
          >
            <input
              @click="handleCheckbox(work.id)"
              type="checkbox"
              :checked="work.status"
              v-model="work.status"
            />
            <span :class="{ completed: work.status }">{{ work.name }}</span>
            <button class="edit-btn">✏️</button>
            <button @click="handleDelete(work.id)" class="delete-btn">
              🗑️
            </button>
          </li>
        </ul>
        <div class="button-container">
          <!-- <p v-if="nitification">{{ nitification }}</p> -->
          <button class="delete-btn">Xóa công việc hoàn thành</button>
          <button class="delete-btn">Xóa tất cả công việc</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { onMounted, ref } from "vue";
const isLoading = ref(true);
const works = ref([]);
const inputName = ref("");
const fetch = async () => {
  try {
    const response = await axios.get("http://localhost:2005/works");
    works.value = response.data;
    console.log(response.data);
  } catch (err) {
    console.log(err);
  } finally {
    setTimeout(() => {
      isLoading.value = false; // Tắt loading sau 4 giây
    }, 2000);
  }
};
onMounted(() => {
  fetch();
});
// const nitification=ref('')
// const ftWork = works.value.filter((work) => work.status === false);
// console.log(111, works.value);

// if(ftWork.length===0){
//   nitification.value="Tất cả công việc đã hoàn thành!!";

// }else{
//   nitification.value='';
// }
const handleCheckbox = async (id) => {
  try {
    const response = await axios.get(`http://localhost:2005/works/${id}`);

    const statusWork = response?.data?.status;

    await axios.patch(`http://localhost:2005/works/${id}`, {
      status: !statusWork ? true : false,
    });

  console.log(statusWork);
    fetch();
  } catch (err) {
    console.log(err);
  }
};

const handleDelete = async (id) => {
  try {
    const workItem = await axios.get(`http://localhost:2005/works/${id}`);
    const confirm = window.confirm(
      `bạn có chắc chắn muốn xóa công việc ${workItem.data.name}?`
    );
    if (confirm) {
      await axios.delete(`http://localhost:2005/works/${id}`);

      fetch();
    } else {
      console.log("lỗi");
    }
  } catch {}
};

const errorMessage = ref("");

const handleAdd = async () => {
  errorMessage.value = "";

  if (!inputName.value.trim()) {
    errorMessage.value = "Tên công việc không được phép để trống.";
    return;
  }

  const exists = works.value.some(
    (work) => work.name === inputName.value.trim()
  );
  if (exists) {
    errorMessage.value = "Tên công việc đã tồn tại. Vui lòng nhập tên khác.";
    return;
  }

  try {
    await axios.post("http://localhost:2005/works", {
      name: inputName.value.trim(),
      status: false,
    });
    alert(" Thêm thành công!");
    inputName.value = "";
    fetch();
  } catch (error) {
    console.error("Đã xảy ra lỗi khi thêm công việc:", error);
    errorMessage.value = "Có lỗi xảy ra. Vui lòng thử lại.";
  }
};
</script>

<style scoped>
.error {
  color: red; /* Màu đỏ cho thông báo lỗi */
  margin: 10px 0;
  font-size: 14px;
}

.loading {
  font-size: 18px;
  color: #007bff;
  margin: 20px 0;
}

body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

.todo-container {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  width: 400px;
  text-align: center;
}

h2 {
  margin-bottom: 20px;
}

.input-container {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

input {
  flex: 1;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
  margin-right: 10px;
}

button {
  padding: 8px 12px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.filter-container {
  display: flex;
  justify-content: space-around;
  margin-bottom: 10px;
}

.filter-container button {
  flex: 1;
  margin-right: 5px;
}

.filter-container button:last-child {
  margin-right: 0;
}

button.active {
  background-color: #28a745;
}

ul {
  list-style: none;
  padding: 0;
  max-height: 330px; /* Giới hạn chiều cao tối đa cho 5 công việc */
  overflow-y: auto; /* Thêm thanh cuộn dọc khi số công việc vượt quá 5 */
  display: flex;
  flex-direction: column;
  align-items: center; /* Căn giữa các công việc */
}

li {
  display: flex;
  gap: 50px;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #ccc;
}

li.completed span {
  text-decoration: line-through;
}

.edit-btn,
.delete-btn {
  background: none;
  border: none;
  cursor: pointer;
}

.edit-btn:hover,
.delete-btn:hover {
  color: red;
}

.button-container {
  margin-top: 20px;
}

.delete-btn {
  background-color: #dc3545;
  margin: 5px 0;
}

.delete-btn:hover {
  background-color: #c82333;
}
</style>
