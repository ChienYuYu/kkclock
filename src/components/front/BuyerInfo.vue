<template>
  <div class="col-12 col-md-10 col-lg-6 mx-auto border p-4 my-5">
    <Form @submit="createOrder" v-slot="{ errors }">
      <h2 class="mb-3">請填寫訂購資料 <span class="ms-3 h6 text-danger">*必填</span></h2>
      <div class="mb-3">
        <label for="email" name="email" class="form-label w-100"
          >Email*
          <Field
            :rules="validateEmail"
            :class="{ 'is-invalid': errors['email'] }"
            type="email"
            class="form-control mt-2"
            id="email"
            name="email"
            aria-describedby="emailHelp"
            placeholder="請輸入Email"
            v-model="form.user.email"
          />
          <ErrorMessage name="email" class="text-danger" />
        </label>
      </div>
      <div class="mb-3">
        <label for="name" class="form-label w-100"
          >收件人姓名*
          <Field
            :rules="validateName"
            :class="{ 'is-invalid': errors['name'] }"
            type="text"
            class="form-control mt-2"
            name="name"
            id="name"
            aria-describedby="emailHelp"
            placeholder="請輸入收件人姓名"
            v-model="form.user.name"
          />
          <ErrorMessage name="name" class="text-danger" />
        </label>
      </div>
      <div class="mb-3">
        <label for="phone" class="form-label w-100"
          >收件人電話*
          <Field
            :rules="validateTel"
            :class="{ 'is-invalid': errors['phone'] }"
            type="tel"
            class="form-control mt-2"
            id="phone"
            name="phone"
            aria-describedby="emailHelp"
            placeholder="請輸入手機ex 0912345678"
            v-model="form.user.tel"
          />
          <ErrorMessage name="phone" class="text-danger" />
        </label>
      </div>
      <div class="mb-3">
        <label for="address" class="form-label w-100"
          >收件人地址*
          <Field
            :rules="validateAddress"
            :class="{ 'is-invalid': errors['address'] }"
            type="text"
            class="form-control mt-2"
            id="address"
            name="address"
            aria-describedby="emailHelp"
            placeholder="請輸入收件人地址"
            v-model="form.user.address"
          />
          <ErrorMessage name="address" class="text-danger" />
        </label>
      </div>
      <div class="mb-3">
        <label for="message" class="form-label w-100">
          留言
          <Field name="message" id="message" class="form-control mt-2" v-model="form.message" />
        </label>
      </div>
      <input type="submit" value="送出訂單 & 付款去" class="btn btn-danger" />
    </Form>
  </div>
</template>

<script>
import { Form, Field, ErrorMessage } from 'vee-validate';
import { ref, inject } from 'vue';
import { useRouter } from 'vue-router';
import emitter from '@/methods/emitter';
import Swal from 'sweetalert2';

export default {
  components: { Form, Field, ErrorMessage },
  setup() {
    const router = useRouter();
    const axios = inject('axios');
    const form = ref({
      user: {
        name: '',
        email: '',
        tel: '',
        address: '',
      },
      message: '',
    });
    const orderId = ref('');

    const createOrder = () => {
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/order`;
      const order = form.value;
      axios.post(api, { data: order })
        .then((res) => {
          if (res.data.success) {
            emitter.emit('updateData');
            orderId.value = res.data.orderId;
            router.push(`checkout/${orderId.value}`);
          }
        })
        .catch(() => {
          Swal.fire({
            title: '似乎有些問題 請稍後再嘗試',
            icon: 'error',
            showConfirmButton: false,
            timer: 2000,
          });
        });
    };

    // 表單驗證規則----------------
    const validateEmail = (value) => {
      if (!value) {
        return '這是必填欄位';
      }

      // if the field is not a valid email
      const regex = /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i;
      if (!regex.test(value)) {
        return '此字段必須是有效的電子郵件';
      }
      return true;
    };

    const validateName = (value) => (!value ? '這是必填欄位' : true);
    // const validateName = (value) => {
    //   if (!value) {
    //     return '這是必填欄位';
    //   }
    //   return true;
    // };

    const validateTel = (value) => {
      if (!value) {
        return '這是必填欄位';
      }
      const phoneNumber = /^(09)[0-9]{8}$/;
      if (!phoneNumber.test(value)) {
        return '需要正確的手機號碼格式';
      }
      return true;
    };

    const validateAddress = (value) => (!value ? '這是必填欄位' : true);
    // const validateAddress = (value) => {
    //   if (!value) {
    //     return '這是必填欄位';
    //   }
    //   return true;
    // };

    return {
      form,
      orderId,
      createOrder,
      validateEmail,
      validateName,
      validateTel,
      validateAddress,
    };
  },

};
</script>
