<script lang="ts">
import { defineComponent, onMounted, onUpdated, reactive, ref, shallowRef, watch } from 'vue';
import axios from 'axios'

interface SearchOrderRequest {
    search:string,
    start_date:string,
    end_date:string,
    page:number,
    per_page:number,
    sort_direction:string,
}

interface SearchOrderResponse {
    request_id:string,
    code:number,
    message:string,
    error_message:string,
    data:OrderDetails,
}

interface OrderDetails {
    total_data:number,
    total_amount:number,
    orders:Order[]
}

interface Order{
    id:string,
    order_name:string,
    product:string,
    company_name:string,
    customer_name:string,
    order_date:string,
    delivered_amount:number,
    total_amount:number
}

const body = shallowRef<SearchOrderRequest>({
    search:"",
    start_date:"",
    end_date:"",
    page:1,
    per_page:5,
    sort_direction:"desc",
})

const response = shallowRef<SearchOrderResponse>({
    request_id:"",
    code:200,
    message:"",
    error_message:"",
    data:{
        total_amount:0,
        total_data:0,
        orders:[] as Order[]
    } as OrderDetails
})

export default defineComponent({
    name:"OrdersComponent",
    setup(){
        const selectedOptions = 5
        const data_per_pages = [5,10,25,50,100]
        const startDateInputType = ref('text')
        const endDateInputType = ref('text')
        const go = 1
        
        
        async function getOrders(body: SearchOrderRequest){
            const url = "http://127.0.0.1:8080/orders"
            const resp = await axios.post<SearchOrderResponse>(url,body)
            .then(function(resp){
                response.value = resp.data
            })
        }
        getOrders(body.value)
        
        const total_pages = Math.ceil(response.value.data.total_data/body.value.per_page)
        const prevCursorClassName = body.value.page == 1 ? 'cursor-not-allowed':''
        const nextCursorClassName = body.value.page == total_pages ? 'cursor-not-allowed':''
        

        return {
            body,
            response,
            startDateInputType,
            endDateInputType,
            go,
            data_per_pages,
            selectedOptions,
            total_pages,
            prevCursorClassName,
            nextCursorClassName
        };
    },
    methods:{
        async getOrders(body: SearchOrderRequest){
            const url = "http://127.0.0.1:8080/orders"
            const resp = await axios.post<SearchOrderResponse>(url,body)
            .then(function(resp){
                response.value = resp.data
            })
        },
        sortByDate(direction: string){
            body.value.sort_direction = direction
            this.getOrders(body.value)
        },
        setStartDateInputType(type:string){
            this.startDateInputType = type
            let start_date = this.$refs.start_date as HTMLInputElement
            body.value.start_date = start_date.value
            this.getOrders(body.value)
        },
        setEndDateInputType(type:string){
            this.endDateInputType = type
            let end_date = this.$refs.end_date as HTMLInputElement
            body.value.end_date = end_date.value
            this.getOrders(body.value)
        },
        updateSearch(){
            let search = this.$refs.search_order as HTMLInputElement
            body.value.search = search.value
            this.getOrders(body.value)
        },
        chooseSelected(){
            body.value.per_page = this.selectedOptions
            this.getOrders(body.value)
            this.total_pages = Math.ceil(response.value.data.total_data/body.value.per_page)
            this.prevCursorClassName = this.total_pages == 1 || this.body.page == 1 ? 'cursor-not-allowed':''
            this.nextCursorClassName = this.body.page == this.total_pages ? 'cursor-not-allowed':''
        },
        showPrev(){

        },
        showNext(){

        }
    }
});
</script>
<template>
    <div class="mx-auto max-w-screen-xl px-4 py-4 lg:px-12">
        <div class="bg-white dark:bg-gray-800 relative shadow-md overflow-hidden">
            <div>
                <div class="flex flex-col md:flex-row items-center justify-between space-y-3 md:space-y-0 md:space-x-4 p-4">
                    <div class="w-full md:w-1/2">
                        <form class="flex items-center">
                            <label for="search-order" class="sr-only">Search</label>
                            <div class="relative w-full">
                                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                                    <svg aria-hidden="true" class="w-5 h-5 text-gray-500 dark:text-gray-400" fill="currentColor" viewbox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                        <path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" />
                                    </svg>
                                </div>
                                <input @blur="updateSearch()" id="search-order" type="text" ref="search_order" name="search" placeholder="search order or product" size="100" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 block w-full pl-10 p-2 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500"/>
                            </div>
                        </form>
                    </div>

                    <div class="w-full md:w-1/2">
                        <label for="start-date-order" class="sr-only">Order Date</label>
                        <span class="h-10 items-center font-normal text-gray-500 dark:text-gray-400">
                            <input id="start-date-order" :type="startDateInputType" ref="start_date" name="start_date" placeholder="start date" size="10" @focus="setStartDateInputType('date')" @blur="setStartDateInputType('text')" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500"/>
                            <span>&nbsp;</span>
                        </span>
                        <label for="end-date-order" class="sr-only">Order Date</label>
                        <span class="h-10 items-center font-normal text-gray-500 dark:text-gray-400">
                            <input id="end-date-order" :type="endDateInputType" ref="end_date" name="end_date" placeholder="end date" size="10" @focus="setEndDateInputType('date')" @blur="setEndDateInputType('text')" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500"/>
                            <span>&nbsp;</span>
                        </span>
                        
                    </div>
                </div>
                
                <div class="flex px-3 flex-col md:flex-row items-center justify-between space-y-3 md:space-y-0 md:space-x-4 p-4">
                    <div class="w-full md:w-1/2">
                        <span class="h-10 items-center font-normal text-gray-500 dark:text-gray-400">
                            Total amount: {{ response?.data.total_amount }}
                        </span>
                    </div>
                </div>
            </div>

            <div>
                <div class="flex px-3">
                    <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
                        <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                            <tr>
                                <th scope="col" class="px-4 py-4">Order name</th>
                                <th scope="col" class="px-4 py-4">Customer company</th>
                                <th scope="col" class="px-4 py-4">Customer name</th>
                                <th scope="col" class="px-4 py-4" @click="sortByDate(body.sort_direction=='desc'?'asc':'desc')">Order date</th>
                                <th scope="col" class="px-4 py-4">Delivered amount</th>
                                <th scope="col" class="px-4 py-4">Total amount</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr class="border-b dark:border-gray-700" v-for="(order,i) in response?.data.orders" :key="i">
                                <td scope="row" class="px-4 py-3 font-medium text-gray-900 whitespace-nowrap dark:text-white">{{ order.order_name }} | {{ order.product }}</td>
                                <td class="px-4 py-3">{{ order.company_name }}</td>
                                <td class="px-4 py-3">{{ order.customer_name }}</td>
                                <td class="px-4 py-3">{{ order.order_date }}</td>
                                <td class="px-4 py-3">{{ order.delivered_amount.toFixed(2) }}</td>
                                <td class="px-4 py-3">{{ order.total_amount.toFixed(2) }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="flex place-content-center items-center px-10 py-10">
                    <span class="text-sm items-center font-normal text-gray-500 dark:text-gray-400">
                        Total <span class="font-semibold text-gray-900 dark:text-white">{{ response?.data.total_data }}</span>
                    </span>
                    <span>&nbsp;</span>

                    <span class="text-sm items-center font-normal text-gray-500 dark:text-gray-400">
                        <select class="form-select rounded-full text-sm font-normal" v-model="selectedOptions" @change="chooseSelected">
                            <option v-for="(val,i) in data_per_pages" :key="i" :value="val" :selected="selectedOptions==val?true:false">{{ val }}</option>
                        </select>
                    </span>
                    <span>&nbsp;</span>
                    
                    <nav aria-label="Table pagination">
                        <ul class="flex items-center -space-x-px text-sm">
                            <li>
                                <a href="#" @click="showPrev" :class="prevCursorClassName" class="flex items-center justify-center px-3 h-8 ms-0 leading-tight text-gray-500 bg-white border border-e-0 border-gray-300 rounded-s-lg hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">
                                    <span class="sr-only">Previous</span>
                                    <svg class="w-2.5 h-2.5 rtl:rotate-180" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 6 10">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 1 1 5l4 4"/>
                                    </svg>
                                </a>
                            </li>
                            <li>
                                <a href="#" ref="current_page_title" class="cursor-not-allowed flex items-center justify-center px-3 h-8 leading-tight text-gray-500 bg-white border border-gray-300 hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">1</a>
                            </li>
                            <li>
                                <a href="#" @click="showNext" :class="nextCursorClassName" class="flex items-center justify-center px-3 h-8 leading-tight text-gray-500 bg-white border border-gray-300 rounded-e-lg hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">
                                    <span class="sr-only">Next</span>
                                    <svg class="w-2.5 h-2.5 rtl:rotate-180" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 6 10">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 9 4-4-4-4"/>
                                    </svg>
                                </a>
                            </li>
                        </ul>
                    </nav>

                    <span>&nbsp;</span>
                    <span class="text-sm items-center font-normal text-gray-500 dark:text-gray-400">
                        Go <input class="form-input rounded-full text-sm font-normal" type="text" inputmode="numeric" pattern="\d*" v-model.lazy="go" name="go" size="2"/>
                    </span>
                </div>
            </div>
        </div>      
    </div>
</template>
<style scoped>
</style>