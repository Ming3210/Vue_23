<!-- src/components/StudentForm.vue -->
<script setup>
import * as z from "zod";
import { ref, watch } from "vue";
import { Button } from "./ui/button";
import { toast } from "./ui/toast";
import { AutoForm } from "./ui/auto-form";
import axios from "axios";

const props = defineProps({
  student: {
    type: Object,
    default: null,
  },
});

const emit = defineEmits(["closeForm", "studentAdded", "studentUpdated"]);

// Định nghĩa schema cho Zod
const schema = z.object({
  name: z.string().describe("Tên học sinh"),
  email: z.string().email().describe("Email"),
  address: z.string().describe("Địa chỉ"),
  phone: z
    .string()
    .min(10, "Số điện thoại phải có ít nhất 10 chữ số")
    .max(11, "Số điện thoại không được quá 11 chữ số")
    .describe("Sđt")
    .refine((val) => /^0\d{9,10}$/.test(val), {
      message: "Số điện thoại phải bắt đầu bằng số 0 và có 10-11 chữ số",
    }),
});

// Khởi tạo formData với dữ liệu rỗng hoặc dữ liệu học sinh nếu có
const formData = ref({
  name: "",
  email: "",
  address: "",
  phone: "",
});

// Khi props.student thay đổi, cập nhật formData
watch(
  () => props.student,
  (newStudent) => {
    if (newStudent) {
      formData.value = {
        name: newStudent.name || "",
        email: newStudent.email || "",
        address: newStudent.address || "",
        phone: newStudent.phone || "",
      };
    } else {
      // Nếu không có student, reset formData
      formData.value = {
        name: "",
        email: "",
        address: "",
        phone: "",
      };
    }
  },
  { immediate: true }
);

const cancel = (e) => {
  e.preventDefault();
  emit("closeForm");
};

const onSubmit = async (values) => {
  if (props.student) {
    await axios.put(
      `http://localhost:8080/students/${props.student.id}`,
      values
    );
    emit("studentUpdated");
    toast({
      title: "Thành công",
      description: "Cập nhật học sinh thành công",
      variant: "custom",
      className: "bg-blue-500 text-white p-5 rounded-md",
    });
  } else {
    await axios.post("http://localhost:8080/students", values);
    emit("studentAdded");
    toast({
      title: "Thành công",
      description: "Thêm học sinh thành công",
      variant: "custom",
      className: "bg-blue-500 text-white p-5 rounded-md",
    });
  }
  emit("closeForm");
};
</script>

<template>
  <form class="w-[50%] bg-white p-20 rounded-md" @submit.prevent>
    <AutoForm
      class="w-2/3 space-y-6"
      :schema="schema"
      :model="formData"
      @submit="onSubmit"
    >
      <Button type="submit">
        {{ props.student ? "Cập nhật" : "Thêm mới" }}
      </Button>
      <Button type="button" @click="cancel">Hủy</Button>
    </AutoForm>
  </form>
  <Toaster />
</template>

<style scoped></style>
