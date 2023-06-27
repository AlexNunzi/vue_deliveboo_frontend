<script>
import {store} from '../store.js';
import OrderTable from '../components/OrderTable.vue';
import axios from 'axios';

export default {
   name: "CheckoutPage",
   data() {
      return {
         store,
         customerName: '',
         customerPhone: '',
         customerEmail: '',
         customerAddress: '',
         nameError: null,
         phoneError: null,
         emailError: null,
         addressError: null,
      }
   },
   components: {
      OrderTable,
   },
   methods: {
         getToken() {
            axios.get(`${this.store.urlApi}client/token`)
            .then(response => {
               this.store.clientBraintreeToken = response.data.token;
               this.brainTree();
            });
         },
         ValidationForm(){

            let button = document.querySelector('#submit-button');
            if(this.customerName != '' && this.customerPhone  != '' && this.customerEmail  != '' && this.customerAddress  != ''){
               button.disabled = false;
            }else{
               button.disabled = true;
            }
         },
         brainTree() {
            let button = document.querySelector('#submit-button');
         
            braintree.dropin.create({
            authorization: this.store.clientBraintreeToken,
            container: '#dropin-container',
            locale: 'it_IT'
            }, (createErr, instance)=> {
            if (createErr) {
               // An error in the create call is likely due to
               // incorrect configuration values or network issues.
               // An appropriate error will be shown in the UI.
               console.log('Create Erros');
               console.error(createErr);
               return;
            }

            button.addEventListener('click', () => {
               instance.requestPaymentMethod((requestPaymentMethodErr, payload) => {
                  if (requestPaymentMethodErr) {
                  // No payment method is available.
                  // An appropriate error will be shown in the UI.
                  console.log('request Payment Method Error')
                  console.error(requestPaymentMethodErr);
                  return;
                  }
                  // Submit payload.nonce to your server

                  let food_ids = [];

                  for(const food in store.cart) {
                     console.log(store.cart[food]);
                     food_ids.push({
                        'id': store.cart[food].foodId,
                        'quantity': store.cart[food].quantity
                     }); 
                  }

                  console.log(food_ids);
                  console.log(payload.nonce);
                  axios.post(`${store.urlApi}client/make-payment`, {
                  'token': payload.nonce,
                  'food_ids': food_ids,
                  'customer_name': this.customerName,                
                  'customer_phone_number': this.customerPhone,
                  'customer_address': this.customerAddress,
                  'customer_email': this.customerEmail
                  }).then(response => {
                     console.log(response);
                  }).catch(error => {
                     console.log(error.response.data.errors);
                     // if(error.response.data.errors.customer_name != null){
                        this.nameError = error.response.data.errors.customer_name;
                     // }
                     // if(error.response.data.errors.customer_phone_number != null){
                        
                     // }
                     this.phoneError = error.response.data.errors.customer_phone_number;
                     this.addressError = error.response.data.errors.customer_address;
                     this.emailError = error.response.data.errors.customer_email;
                  });
               });
            });
         });
      }
   },
   mounted() {
      this.getToken();
   }
}
</script>

<template>
   <div class="pt-5">
      <OrderTable />
   </div>
   <div v-show="!store.cartIsEmpty()" class="row row-cols-md-2">
      <!-- <form id="payment-form" action="/success" method="get">
         <div id="dropin-container"></div>
         <input type="submit" />
      </form> -->
      <form method="POST" class="mt-5 py-2 bg-white border h-100" @click="changeFormValue" >
         <div class="mb-3">
            <div class="d-flex">
               <div class="w-75">
                  <label for="customer_name" class="form-label">Nome:</label>
                  <input type="text" class="form-control" :class="nameError != null ? 'is-invalid' : ''" id="customer_name" name="customer_name" placeholder="Mario Rossi" @keyup="ValidationForm()"  v-model="customerName" required>
                  <div v-if="nameError != null"  class="invalid-feedback">
                     <p v-for="element in nameError">{{ element }}</p>
                  </div>
               </div>
               <div class="ms-5">
                  <label for="customer_phone_number" class="form-label">Telefono:</label>
                  <input type="tel" class="form-control" :class="phoneError != null ? 'is-invalid' : ''" id="customer_phone_number" name="customer_phone_number" @keyup="ValidationForm()" placeholder="012-3456789"  v-model="customerPhone" required>
                  <div v-if="phoneError != null" class="invalid-feedback">
                     <p v-for="element in phoneError">{{ element }}</p>
                  </div>
               </div>
            </div>
         </div>
         <div class="mb-3">
            <label for="customer_address" class="form-label">Indirizzo:</label>
            <input type="text" class="form-control" :class="addressError != null ? 'is-invalid' : ''" id="customer_address" name="customer_address" @keyup="ValidationForm()" placeholder="Via Le dita dal Naso 5"  v-model="customerAddress" required>
            <div v-if="addressError != null" class="invalid-feedback">
               <p v-for="element in addressError">{{ element }}</p>
            </div>
         </div>
         <div class="mb-3">
            <label for="customer_email" class="form-label">Email</label>
            <input type="email" class="form-control" :class="emailError != null ? 'is-invalid' : ''" id="customer_email" name="customer_email" @keyup="ValidationForm()" placeholder="m.rossi@deliveboo.it"  v-model="customerEmail" required>
            <div v-if="emailError != null" class="invalid-feedback">
               <p v-for="element in emailError">{{ element }}</p>
            </div>
         </div>
      </form>
      <div class="mt-3">
         <div id="dropin-container"></div>
         <button  id="submit-button" type="submit" class="btn btn-primary" disabled >Salva</button>
         <!-- <button class="button btn btn-outline-success">Purchase</button> -->
         <ul>
            <li>Visa: 4111111111111111</li>
            <li>Mastercard: 5555555555554444</li>
            <!-- <li>American Express: 378282246310005</li>
            <li>Discover: 6011111111111117</li>
            <li>JCB: 3530111333300000</li> -->
         </ul>
         <p>data: 01/25</p>
      </div>
   </div>
</template>

<style lang="scss" scoped>
@use "../styles/general.scss";

//brainTree
.braintree-heading, .braintree-placeholder, .braintree-sheet__header .braintree-sheet__header-label{
   display: none;
}
</style>