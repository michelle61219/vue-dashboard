<template>
    <LoadingOverlay :active="isLoading"></LoadingOverlay>
    <div class="container">
        <div class="row mt-4">
            <div class="col-md-7">
                <!-- product list -->
                <table class="table align-middle">
                    <thead>
                        <tr>
                            <th>圖片</th>
                            <th>商品名稱</th>
                            <th>價格</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="item in products" :key="item.id" v-show="item.is_enabled">
                            <td style="width: 200px">
                                <div style=" height: 180px; background-size: cover;background-position: center;"
                                    :style="{ backgroundImage: `url(${item.imageUrl})` }"></div>
                            </td>
                            <td>
                                <h4>{{ item.title }}</h4>
                            </td>
                            <td>
                                <div v-if="!(item.origin_price == item.price)">
                                    <del class="h6" v-if="item.price">原價 {{ item.origin_price }} 元</del>
                                    <div class="h5" v-if="item.price"> 現在只要 {{ item.price }} 元</div>
                                </div>
                                <div class="h5" v-else>{{ item.origin_price }}元</div>
                            </td>
                            <td>
                                <div class="btn-group btn-group-sm">
                                    <button type="button" class="btn btn-outline-secondary" @click="getProduct(item.id)">
                                        查看更多
                                    </button>
                                    <button type="button" class="btn btn-outline-danger"
                                        :disabled="this.status.loadingItem === item.id" @click="addCart(item.id)">
                                        <div v-if="this.status.loadingItem === item.id"
                                            class="spinner-grow text-danger spinner-grow-sm" role="status">
                                            <span class="visually-hidden">Loading...</span>
                                        </div>
                                        加到購物車
                                    </button>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <!-- shopping cart -->
            <div class="col-md-5">
                <div class="sticky-top">
                    <table class="table align-middle">
                        <thead>
                            <tr>
                                <th></th>
                                <th>品名</th>
                                <th style="width: 110px">數量</th>
                                <th>單價</th>
                            </tr>
                        </thead>
                        <tbody>
                            <template v-if="cart.carts">
                                <tr v-for="item in cart.carts" :key="item.id">
                                    <td>
                                        <button type="button" class="btn btn-outline-danger btn-sm"
                                            :disabled="status.loadingItem === item.id" @click="removeCartItem(item.id)">
                                            <i class="bi bi-x"></i>
                                        </button>
                                    </td>
                                    <td>
                                        {{ item.product.title }}
                                        <div class="text-success" v-if="item.coupon">
                                            已套用優惠券
                                        </div>
                                    </td>
                                    <td>
                                        <div class="input-group input-group-sm">
                                            <input type="number" class="form-control" v-model.number="item.qty" min="1"
                                                @change="updateCart(item)">
                                            <div class="input-group-text">/ {{ item.product.unit }}</div>
                                        </div>
                                    </td>
                                    <td class="text-end">
                                        <small v-if="cart.final_total !== cart.total" class="text-success">折扣價：</small>
                                        {{ $filters.currency(item.final_total) }}
                                    </td>
                                </tr>
                            </template>
                        </tbody>
                        <tfoot>
                            <tr>
                                <td colspan="3" class="text-end">總計</td>
                                <td class="text-end">{{ $filters.currency(cart.total) }}</td>
                            </tr>
                            <tr v-if="cart.final_total !== cart.total">
                                <td colspan="3" class="text-end text-success">折扣價</td>
                                <td class="text-end text-success">{{ $filters.currency(cart.final_total) }}</td>
                            </tr>
                        </tfoot>
                    </table>
                    <div class="input-group mb-3 input-group-sm">
                        <input type="text" class="form-control" v-model="coupon_code" placeholder="請輸入優惠碼">
                        <div class="input-group-append">
                            <button class="btn btn-outline-secondary" type="button" @click="addCouponCode">
                                套用優惠碼
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!-- 訂單表單 -->
        <div class="my-5 row justify-content-center">
            <vForm class="col-md-6" v-slot="{ errors }" @submit="createOrder">
                <div class="mb-3">
                    <label for="email" class="form-label">Email</label>
                    <vField id="email" name="email" type="email" class="form-control"
                        :class="{ 'is-invalid': errors['email'] }" placeholder="請輸入 Email" rules="email|required"
                        v-model="form.user.email"></vField>
                    <vErrorMessage name="email" class="invalid-feedback"></vErrorMessage>
                </div>

                <div class="mb-3">
                    <label for="name" class="form-label">收件人姓名</label>
                    <vField id="name" name="姓名" type="text" class="form-control" :class="{ 'is-invalid': errors['姓名'] }"
                        placeholder="請輸入姓名" rules="required" v-model="form.user.name"></vField>
                    <vErrorMessage name="姓名" class="invalid-feedback"></vErrorMessage>
                </div>

                <div class="mb-3">
                    <label for="tel" class="form-label">收件人電話</label>
                    <vField id="tel" name="電話" type="tel" class="form-control" :class="{ 'is-invalid': errors['電話'] }"
                        placeholder="請輸入電話" :rules="isPhone" v-model="form.user.tel"></vField>
                    <vErrorMessage name="電話" class="invalid-feedback"></vErrorMessage>
                </div>

                <div class="mb-3">
                    <label for="address" class="form-label">收件人地址</label>
                    <vField id="address" name="地址" type="text" class="form-control" :class="{ 'is-invalid': errors['地址'] }"
                        placeholder="請輸入地址" rules="required" v-model="form.user.address"></vField>
                    <vErrorMessage name="地址" class="invalid-feedback"></vErrorMessage>
                </div>

                <div class="mb-3">
                    <label for="message" class="form-label">留言</label>
                    <textarea name="" id="message" class="form-control" cols="30" rows="10"
                        v-model="form.message"></textarea>
                </div>
                <div class="text-end">
                    <button class="btn btn-danger">送出訂單</button>
                </div>
            </vForm>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            products: [],
            //product: { }, // 單一選項(detailed product page)
            status: {
                loadingItem: "", // 對應品項id, prevents overclicking addCart button
            },
            form: {
                user: {
                    name: '',
                    email: '',
                    tel: '',
                    address: '',
                },
                message: '',
            },
            cart: {},
            coupon_code: "",
        };
    },
    inject: ["pushMessageState"],
    methods: {
        getProducts() {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/admin/products/all`;
            this.isLoading = true;
            this.$http.get(url).then((response) => {
                this.products = response.data.products;
                // console.log('products:', response);
                this.isLoading = false;
            });
        },
        getProduct(id) {
            this.$router.push(`/user/product/${id}`);
        },
        addCart(id) {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`;
            this.status.loadingItem = id;
            const cart = {
                product_id: id,
                qty: 1,
            };
            this.$http.post(url, { data: cart }).then((res) => {
                this.status.loadingItem = " ";
                console.log(res);
                this.getCart();
            });
        },
        getCart() {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`;
            this.isLoading = true;
            this.$http.get(url).then((response) => {
                // data.data.carts (All products in cart) & finall_total cost
                this.cart = response.data.data;
                this.isLoading = false;
            });
        },
        // 更新購物車
        updateCart(item) {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart/${item.id}`;
            this.isLoading = true;
            this.status.loadingItem = item.id;
            const cart = {
                product_id: item.product_id,
                qty: item.qty,
            };
            this.$http.put(url, { data: cart }).then((res) => {
                console.log(res);
                this.status.loadingItem = " ";
                this.getCart();
            });
        },
        // 刪除購物車裡不要的商品
        removeCartItem(id) {
            this.status.loadingItem = id;
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart/${id}`;
            this.isLoading = true;
            this.$http.delete(url).then((response) => {
                this.pushMessageState(response, "移除購物車品項");
                this.status.loadingItem = "";
                this.getCart();
                this.isLoading = false;
            });
        },
        // 套用優惠券結帳
        addCouponCode() {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/coupon`;
            const coupon = {
                code: this.coupon_code,
            };
            this.isLoading = true;
            this.$http.post(url, { data: coupon }).then((response) => {
                this.pushMessageState(response, '加入優惠券');
                console.log(response);
                this.getCart();
                this.isLoading = false;
            });
        },
        //建立訂購單
        createOrder() {
            const url = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/order`;
            const order = this.form;
            this.$http.post(url, { data: order }).then((res) => {
                console.log(res);
            });
        },
        //檢視電話號碼
        isPhone(value) {
            const phoneNumber = /^(09)[0-9]{8}$/
            return phoneNumber.test(value) ? true : '需要正確的電話號碼'
        }
    },
    created() {
        this.getProducts();
        this.getCart();
    },
};
</script>
