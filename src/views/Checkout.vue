<template>
    <div class="page-checkout">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Podsumowanie</h1>
            </div>

            <div class="column is-12 box">
                <table class="table is-fullwidth">
                    <thead>
                        <tr>
                            <th>Produkt</th>
                            <th>Rozmiar</th>
                            <th>Cena</th>
                            <th>Ilość</th>
                            <th>Suma</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr
                            v-for="item in cart.items"
                            v-bind:key="item.product.id"
                        >
                            <td>{{ item.product.name }}</td>
                            <td>{{ item.selected }}</td>
                            <td>PLN {{ item.product.price }}</td>
                            <td>{{ item.quantity }}</td>
                            <td>PLN {{ item.product.price * item.quantity }}</td>
                        </tr>
                    </tbody>

                    <tfoot>
                        <tr>
                            <td colspan="2">Suma</td>
                            <td>{{ cartTotalLength }}</td>
                            <td>PLN {{ cartTotalPrice.toFixed(2) }}</td>
                        </tr>
                    </tfoot>
                </table>
            </div>

            <div class="column is-12 box">
                <h2 class="subtitle">Informacje do zamówienia</h2>

                <p class="has-text-grey mb-4">* Wszystkie pola muszą zostać wypełnione</p>

                <div class="columns is-multiline">
                    <div class="column is-6">
                        <div class="field">
                            <label>Imię*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="first_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>Nazwisko*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="last_name">
                            </div>
                        </div>

                        <div class="field">
                            <label>E-mail*</label>
                            <div class="control">
                                <input type="email" class="input" v-model="email">
                            </div>
                        </div>

                        <div class="field">
                            <label>Telefon kontaktowy*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="phone">
                            </div>
                        </div>
                    </div>

                    <div class="column is-6">
                        <div class="field">
                            <label>Adres*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="address">
                            </div>
                        </div>

                        <div class="field">
                            <label>Kod-pocztowy*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="zipcode">
                            </div>
                        </div>

                        <div class="field">
                            <label>Miasto*</label>
                            <div class="control">
                                <input type="text" class="input" v-model="place">
                            </div>
                        </div>

                        <br>

                        <div class="field">
                            <label>Sposób dostawy*</label>
                        <div class="control">
                        <select v-model="shipment">
                          <option disabled value="">Proszę wybrać sposób dostawy</option>
                          <option>Paczkomat Inpost-10zł</option>
                          <option>Kurier Inpost 48h-20zł</option>
                          <option>Odbiór osobisty</option>
                        </select>

                    </div>
                       </div>

                       <div class="field">
                            <label>Sposób płatności*</label>
                        <div class="control">
                        <select v-model="payment">
                          <option disabled value="">Proszę wybrać sposób płatności</option>
                          <option>Przelew krajowy</option>
                          <option>Blik</option>
                          <option>Płatność przy odbiorze</option>
                        </select>



                    </div>
                    </div>



                    </div>
                </div>

                <div class="notification is-danger mt-4" v-if="errors.length">
                    <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
                </div>

                <hr>

                <div id="card-element" class="mb-5"></div>

                <template v-if="cartTotalLength">
                    <hr>

                    <button class="button is-dark" @click="submitForm">Przejdź do płatności</button>
                </template>


                <hr>
                <h1 class="title">Wybór paczkomatu</h1>
                <div class="column is-12 box" id="easypack-map" style="height: 150px;
                width: 100%;">
                <div class="hero is-small is-black mb-4">
                </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'

            window.easyPackAsyncInit = function () {
            easyPack.init({
              mapType: 'osm',
              searchType: 'osm',
              defaultLocale: 'pl',
                points: {
                    types: ['parcel_locker']
                },
                map: {
                    defaultLocation: [51.507351, -0.127758],
                    initialTypes: ['parcel_locker']
                }
            });
            var map = easyPack.mapWidget('easypack-map', function(point) {
              console.log(point), alert('Wybrano paczkomat');
              localStorage.setItem('paczkomat', JSON.stringify(point));
            });
          };


export default {
    name: 'Checkout',
    data() {
        return {
            shipment: '',
            payment: '',
            cart: {
                items: []
            },
            stripe: {},
            card: {},
            first_name: '',
            last_name: '',
            email: '',
            phone: '',
            address: '',
            zipcode: '',
            place: '',
            errors: []
        }
    },

     watch: {
       shipment(shipment) {
            localStorage.shipment = shipment;
       },
       payment(payment) {
            localStorage.payment = payment;
       },
    },
    mounted() {
        document.title = 'Zamówienie | Asiola Butik'

        this.cart = this.$store.state.cart

        if (this.cartTotalLength > 0) {
            this.stripe = Stripe('pk_test_51ImOgWIcogfCGBZIumpr9o1QfzZwihd0CWoyWNKMK1QZg9CfcCCod9ujgT4mFyygFrVhnzRDOnsrUo4l2aSgDOQ400HYmUsz6K')
            const elements = this.stripe.elements();
            this.card = elements.create('card', { hidePostalCode: true })
            this.card.mount('#card-element')
        }

        this.getLocalStorage
        if (localStorage.payment) {
            this.payment = localStorage.payment;
            this.shipment = localStorage.shipment;
        }

        let recaptchaScript = document.createElement('script')
            recaptchaScript.setAttribute('src', 'https://geowidget.easypack24.net/js/sdk-for-javascript.js')
            document.head.appendChild(recaptchaScript);
            let styles = document.createElement('link')
            styles.setAttribute('href', "https://geowidget.easypack24.net/css/easypack.css");
            styles.setAttribute('height', '200px')
            styles.setAttribute('rel', 'stylesheet');
            document.head.appendChild(styles);




    },
    methods: {

        getItemTotal(item) {
            return item.quantity * item.product.price * item.selected
        },
        submitForm() {
            this.errors = []
            if (this.first_name === '') {
                this.errors.push('Nie uzupełniono pola Imię!')
            }
            if (this.last_name === '') {
                this.errors.push('Nie uzupełniono pola Nazwisko!')
            }
            if (this.email === '') {
                this.errors.push('Nie uzupełniono pola E-mail!')
            }
            if (this.phone === '') {
                this.errors.push('Nie uzupełniono pola Telefon kontaktowy!')
            }
            if (this.address === '') {
                this.errors.push('Nie uzupełniono pola Adres!')
            }
            if (this.zipcode === '') {
                this.errors.push('Nie uzupełniono pola Kod-pocztowy!')
            }
            if (this.place === '') {
                this.errors.push('Nie uzupełniono pola Miasto!')
            }

            if (!this.errors.length) {
                this.$store.commit('setIsLoading', true)
                this.stripe.createToken(this.card).then(result => {
                    if (result.error) {
                        this.$store.commit('setIsLoading', false)
                        this.errors.push('Coś poszło nie tak. Proszę spróbować ponownie')
                        console.log(result.error.message)
                    } else {
                        this.stripeTokenHandler(result.token)
                    }
                })
            }
        },
        async stripeTokenHandler(token) {
            const items = []
            for (let i = 0; i < this.cart.items.length; i++) {
                const item = this.cart.items[i]
                const obj = {
                    product: item.product.id,
                    quantity: item.quantity,
                    size: item.selected,
                    price: item.product.price * item.quantity,
                    payment: this.payment,
                    shipment: this.shipment,
                }
                items.push(obj)
            }
            const data = {
                'first_name': this.first_name,
                'last_name': this.last_name,
                'email': this.email,
                'address': this.address,
                'zipcode': this.zipcode,
                'place': this.place,
                'phone': this.phone,
                'items': items,
                'stripe_token': token.id
            }
            await axios
                .post('/api/v1/checkout/', data)
                .then(response => {
                    this.$store.commit('clearCart')
                    this.$router.push('/cart/success')
                })
                .catch(error => {
                    this.errors.push('Coś poszło nie tak. Proszę spróbować ponownie')
                    console.log(error)
                })
                this.$store.commit('setIsLoading', false)
        }
    },
   computed: {
        cartTotalPrice() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.product.price * curVal.quantity
            }, 0)
        },
        cartTotalLength() {
            return this.cart.items.reduce((acc, curVal) => {
                return acc += curVal.quantity
            }, 0)
        }
    }
}
</script>