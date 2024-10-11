<template>
  <div>
    <!-- Modal StudentForm -->
    <div
      v-if="form || editForm"
      class="flex items-center justify-center bg-opacity-50 absolute w-[100vw] h-[100vh] bg-slate-900"
    >
      <AddForm
        :student="currentStudent"
        @closeForm="closeForm"
        @studentAdded="handleStudentAdded"
        @studentUpdated="handleStudentUpdated"
      />
    </div>

    <Toaster />
    <nav class="nav h-[80px] leading-[80px]">
      <div class="flex justify-between mx-7 items-center">
        <h2 class="text-white">Quản lý sinh viên</h2>
        <Button @click="openForm" class="bg-green-400">
          <b class="mr-2 bg-white rounded-full w-[20px] h-[20px] text-black"
            >+</b
          >
          Thêm mới sinh viên
        </Button>
      </div>
    </nav>

    <main>
      <div class="w-full">
        <div class="flex gap-2 items-center py-4">
          <Input
            class="max-w-sm"
            placeholder="Filter emails..."
            v-model="inputV"
          />
        </div>
        <button @click="search">Search</button>
        <!-- Students Table -->
        <div class="rounded-md border overflow-x-auto">
          <table class="min-w-full divide-y divide-gray-200">
            <!-- Table Header -->
            <thead class="bg-gray-50">
              <tr>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Họ và tên
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Email
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Địa chỉ
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Sđt
                </th>
                <th
                  class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Tùy chọn
                </th>
              </tr>
            </thead>

            <!-- Table Body -->
            <tbody class="bg-white divide-y divide-gray-200">
              <tr
                v-for="student in students"
                :key="student.id"
                class="hover:bg-gray-100"
              >
                <td class="px-6 py-4 whitespace-nowrap">{{ student.name }}</td>
                <td class="px-6 py-4 whitespace-nowrap">{{ student.email }}</td>
                <td class="px-6 py-4 whitespace-nowrap">
                  {{ student.address }}
                </td>
                <td class="px-6 py-4 whitespace-nowrap">{{ student.phone }}</td>
                <td class="px-6 py-4 whitespace-nowrap flex space-x-2">
                  <Button
                    variant="outline"
                    size="sm"
                    @click="editStudent(student)"
                  >
                    Sửa
                  </Button>
                  <Button
                    variant="destructive"
                    size="sm"
                    @click="del(student.id)"
                  >
                    Xóa
                  </Button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="flex items-center justify-end space-x-2 py-4">
          <div class="flex-1 text-sm text-muted-foreground"></div>
          <div class="space-x-2">
            <Button variant="outline" size="sm">Previous</Button>
            <Button variant="outline" size="sm">Next</Button>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import axios from "axios";
import Toaster from "./components/ui/toast/Toaster.vue";
import { useToast } from "@/components/ui/toast/use-toast";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { onMounted, ref } from "vue";
import AddForm from "./components/AddForm.vue";

const form = ref(false); // Trạng thái để hiển thị StudentForm cho thêm
const editForm = ref(false); // Trạng thái để hiển thị StudentForm cho chỉnh sửa
const inputV = ref("");
const students = ref([]);
const currentStudent = ref(null); // Lưu trữ học sinh hiện tại đang chỉnh sửa
const { toast } = useToast();

// Hàm lấy dữ liệu học sinh từ API
const getData = async () => {
  const response = await axios.get("http://localhost:8080/students");
  students.value = response.data;
};

// Hàm xóa học sinh
const del = async (id) => {
  const confirmDelete = window.confirm("Bạn có chắc chắn muốn xóa?");
  if (!confirmDelete) return;

  await axios.delete(`http://localhost:8080/students/${id}`);
  getData();
  toast({
    title: "Thành công",
    description: "Xóa học sinh thành công",
    variant: "success",
  });
};

// Hàm mở StudentForm ở chế độ thêm
const openForm = () => {
  form.value = true;
  currentStudent.value = null;
  editForm.value = false;
};

// Hàm mở StudentForm ở chế độ chỉnh sửa
const editStudent = (student) => {
  currentStudent.value = student;
  editForm.value = true;
  form.value = false;
};

const search = async () => {
  const res = await axios.get(
    `http://localhost:8080/students?email_like=${inputV.value}`
  );
  students.value = res.data;
};

// Hàm xử lý khi thêm học sinh thành công
const handleStudentAdded = () => {
  getData(); // Lấy lại danh sách học sinh
  toast({
    title: "Thành công",
    description: "Thêm học sinh thành công",
    variant: "success",
  });
};

// Hàm xử lý khi cập nhật học sinh thành công
const handleStudentUpdated = () => {
  getData(); // Lấy lại danh sách học sinh
  toast({
    title: "Thành công",
    description: "Cập nhật học sinh thành công",
    variant: "success",
  });
};

// Hàm đóng StudentForm
const closeForm = () => {
  form.value = false;
  editForm.value = false;
  currentStudent.value = null;
  toast({
    title: "Hủy bỏ",
    description: "Form đã được đóng",
    variant: "destructive",
  });
};

// Khởi tạo dữ liệu khi component được mount
onMounted(() => {
  getData();
});
</script>

<style scoped>
nav {
  background-color: #435d7d;
}
.success {
  background-color: #28a745;
}
</style>
