<script lang="ts">
import { defineComponent, ref, shallowRef } from 'vue';

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
    company:string,
    customer:string,
    order_date:string,
    delivered_amount:number,
    total_amount:number
}

export default defineComponent({
    name:"OrdersComponent",
    setup(){
        const per_pages = [5,10,25,50,100]
        const startDateInputType = ref('text')
        const endDateInputType = ref('text')
        const go = 1
        
        const request = shallowRef<SearchOrderRequest>({
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
                orders:[
                    {id:"", order_name:"PO #004-P", product:"", company:"Pupkin & Co", customer:"Petr Petrovich", order_date:"2020-02-02T15:34:12+11:00", delivered_amount:0, total_amount:910.736},
                    {id:"", order_name:"PO #004-I", product:"", company:"Roga & Kopyta", customer:"Ivan Ivanovich", order_date:"2020-02-02T15:34:12+11:00", delivered_amount:0, total_amount:263.35}
                ] as Order[]
            } as OrderDetails
        })
        
       return {
            request,
            response,
            startDateInputType,
            endDateInputType,
            go,
            per_pages
        };
    },
    methods:{
        sortByDate(direction: string){
            this.request.sort_direction = direction
        },
        setStartDateInputType(type:string){
            this.startDateInputType = type
        },
        setEndDateInputType(type:string){
            this.endDateInputType = type
            console.log(this.request)
        },
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
                                <input id="search-order" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 block w-full pl-10 p-2 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500" type="text" v-model.lazy="request.search" name="search" placeholder="search order or product" size="100"/>
                            </div>
                        </form>
                    </div>

                    <div class="w-full md:w-1/2">
                        <label for="start-date-order" class="sr-only">Order Date</label>
                        <span class="h-10 items-center font-normal text-gray-500 dark:text-gray-400">
                            <input id="start-date-order" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500" :type="startDateInputType" v-model.lazy="request.start_date" name="start_date" placeholder="start date" size="10" @focus="setStartDateInputType('date')" @blur="setStartDateInputType('text')"/>
                            <span>&nbsp;</span>
                        </span>
                        <label for="end-date-order" class="sr-only">Order Date</label>
                        <span class="h-10 items-center font-normal text-gray-500 dark:text-gray-400">
                            <input id="end-date-order" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-primary-500 dark:focus:border-primary-500" :type="endDateInputType" v-model.lazy="request.end_date" name="end_date" placeholder="end date" size="10" @focus="setEndDateInputType('date')" @blur="setEndDateInputType('text')"/>
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
                                <th scope="col" class="px-4 py-4" @click="sortByDate(request.sort_direction=='desc'?'asc':'desc')">Order date</th>
                                <th scope="col" class="px-4 py-4">Delivered amount</th>
                                <th scope="col" class="px-4 py-4">Total amount</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr class="border-b dark:border-gray-700" v-for="(order,i) in response?.data.orders" :key="i">
                                <td scope="row" class="px-4 py-3 font-medium text-gray-900 whitespace-nowrap dark:text-white">{{ order.order_name }} {{ order.product }}</td>
                                <td class="px-4 py-3">{{ order.company }}</td>
                                <td class="px-4 py-3">{{ order.customer }}</td>
                                <td class="px-4 py-3">{{ order.order_date }}</td>
                                <td class="px-4 py-3">{{ order.delivered_amount.toFixed(2) }}</td>
                                <td class="px-4 py-3">{{ order.total_amount.toFixed(2) }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="flex place-content-center items-center px-10 py-10">
                    <span class="text-sm h-4 items-center font-normal text-gray-500 dark:text-gray-400">
                        Total <span class="font-semibold text-gray-900 dark:text-white">{{ response?.data.total_data }}</span>
                    </span>
                    <span>&nbsp;</span>

                    <span class="text-sm h-4 items-center font-normal text-gray-500 dark:text-gray-400">
                        <select class="form-select rounded-full text-sm h-4 font-normal">
                            <option v-for="(val,i) in per_pages" :key="i" value="{{ val }}">{{ val }}</option>
                        </select>
                    </span>
                    <span>&nbsp;</span>
                    
                    <nav aria-label="Table pagination">
                        <ul class="flex items-center -space-x-px h-8 text-sm">
                            <li>
                                <a href="#" class="flex items-center justify-center px-3 h-8 ms-0 leading-tight text-gray-500 bg-white border border-e-0 border-gray-300 rounded-s-lg hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">
                                    <span class="sr-only">Previous</span>
                                    <svg class="w-2.5 h-2.5 rtl:rotate-180" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 6 10">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 1 1 5l4 4"/>
                                    </svg>
                                </a>
                            </li>
                            <li>
                                <a href="#" class="flex items-center justify-center px-3 h-8 leading-tight text-gray-500 bg-white border border-gray-300 hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">1</a>
                            </li>
                            <li>
                                <a href="#" class="flex items-center justify-center px-3 h-8 leading-tight text-gray-500 bg-white border border-gray-300 rounded-e-lg hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white">
                                    <span class="sr-only">Next</span>
                                    <svg class="w-2.5 h-2.5 rtl:rotate-180" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 6 10">
                                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 9 4-4-4-4"/>
                                    </svg>
                                </a>
                            </li>
                        </ul>
                    </nav>

                    <span>&nbsp;</span>
                    <span class="text-sm h-4 items-center font-normal text-gray-500 dark:text-gray-400">
                        Go <input class="form-input rounded-full text-sm h-4 font-normal" type="text" inputmode="numeric" pattern="\d*" v-model.lazy="go" name="go" size="2"/>
                    </span>
                </div>
            </div>
        </div>      
    </div>
</template>
<style scoped>
</style>