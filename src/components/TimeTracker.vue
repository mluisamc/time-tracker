<template>
  <div class="flex bg-gray-100 rounded-full p-3" v-if="is_data_fetched">
    <div class="flex-initial w-12 text-black p-3">
      <span>{{stopwatch.hours}}</span>:<span>{{stopwatch.minutes}}</span>:<span>{{stopwatch.seconds}}</span>
    </div>
    <div class="flex-initial w-22 text-slate-300 p-3">
      /<span>{{currentHours}}</span>:<span>{{currentMinutes}}</span>:<span>{{currentSeconds}}</span>
    </div>
    <button v-show="workStatus != 'online'" @click="stopwatch.start()" class="bg-green-300 hover:bg-green-400 text-white font-bold py-2 px-4 rounded-full flex-initial w-32 m-1">
      Entrar
    </button>
    <button v-show="workStatus == 'online'" @click="stopwatch.pause()" class="bg-gray-300 hover:bg-gray-400 text-white font-bold py-2 px-4 rounded-full flex-initial w-32 m-1">
      Pausar
    </button>
    <button v-show="workStatus == 'online'" @click="stopwatch.reset()" class="bg-red-400 hover:bg-red-500 text-white font-bold py-2 px-4 rounded-full flex-initial w-32 m-1">
      Salir
    </button>
    <div class="flex -space-x-2 overflow-hidden flex-initial w-12 m-1">
      <img class="inline-block h-8 w-8 rounded-full" src="https://images.unsplash.com/photo-1550525811-e5869dd03032?ixlib=rb-1.2.1&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80" alt="" />
      <span class="top-0 left-7 absolute w-3.5 h-3.5 bg-green-400 rounded-full"></span>
    </div>

    <Menu as="div" class="relative inline-block text-left flex-initial w-32 m-1">
    <div>
      <MenuButton class="inline-flex w-full justify-center gap-x-1.5 rounded-md bg-grey px-3 py-2 text-sm font-bold text-gray-900 hover:bg-gray-50">
        Kelly Pierce
        <ChevronDownIcon class="-mr-1 h-5 w-5 text-gray-400" aria-hidden="true" />
      </MenuButton>
    </div>

    <transition enter-active-class="transition ease-out duration-100" enter-from-class="transform opacity-0 scale-95" enter-to-class="transform opacity-100 scale-100" leave-active-class="transition ease-in duration-75" leave-from-class="transform opacity-100 scale-100" leave-to-class="transform opacity-0 scale-95">
      <MenuItems class="absolute right-0 z-10 mt-5 w-56 origin-top-right divide-y divide-gray-100 rounded-md bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none">
        <div class="py-1"> 
          <Menu>         
            <MenuButton class="inline-flex w-full gap-x-1.5 rounded-md bg-grey px-3 py-2 text-sm text-gray-900 hover:bg-gray-50">
              <ChevronLeftIcon class="-mr-1 h-5 w-5 text-gray-400" aria-hidden="true" />
              Mis cuentas
            </MenuButton>
            <transition enter-active-class="transition ease-out duration-100" enter-from-class="transform opacity-0 scale-95" enter-to-class="transform opacity-100 scale-100" leave-active-class="transition ease-in duration-75" leave-from-class="transform opacity-100 scale-100" leave-to-class="transform opacity-0 scale-95">
              <MenuItems class="absolute right-20 z-10 w-56 origin-top-left divide-y divide-gray-100 rounded-md bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none">
                <div class="py-1">
                  <MenuItem v-slot="{ active }">
                    <a href="#" :class="[active ? 'bg-gray-100 text-gray-900' : 'text-gray-700', 'block px-4 py-2 text-sm']" class="font-bold">
                      Sesame 1 
                      <span class="font-bold text-slate-400">Kelly Pierce </span> 
                      <br> 
                      <span class="text-slate-300">Hoy llevas 00:00</span>
                    </a>
                  </MenuItem>
                </div>
                <div class="py-1">
                  <MenuItem v-slot="{ active }">
                    <a href="#" :class="[active ? 'bg-gray-100 text-gray-900' : 'text-gray-700', 'block px-4 py-2 text-sm']" class="font-bold">
                      Sesame 2
                      <span class="font-bold text-slate-400">Kelly Pierce </span> 
                      <br> 
                      <span class="text-slate-300">Hoy llevas 00:00</span>
                    </a>
                  </MenuItem>
                </div>
              </MenuItems>
            </transition>
          </Menu>
        </div>
        <div class="py-1">
          <MenuItem v-slot="{ active }">
            <a href="#" :class="[active ? 'bg-gray-100 text-gray-900' : 'text-gray-700', 'block px-4 py-2 text-sm']">Vista empleado</a>
          </MenuItem>
        </div>
        <div class="py-1">
          <MenuItem v-slot="{ active }">
            <a href="#" :class="[active ? 'bg-gray-100 text-gray-900' : 'text-gray-700', 'block px-4 py-2 text-sm']">Mi perfil</a>
          </MenuItem>
        </div>
        <div class="py-1">
          <MenuItem v-slot="{ active }">
            <a href="#" :class="[active ? 'bg-gray-100 text-gray-900' : 'text-gray-700', 'block px-4 py-2 text-sm']">Cerrar sesión</a>
          </MenuItem>
        </div>
      </MenuItems>
    </transition>
  </Menu>
  </div>
</template>

<script setup>
  import { Menu, MenuButton, MenuItem, MenuItems } from '@headlessui/vue'
  import { ChevronDownIcon, ChevronLeftIcon } from '@heroicons/vue/20/solid'
</script>

<script>
import { useStopwatch } from 'vue-timer-hook';
  import axios from 'axios'
  import moment from 'moment';

  const reqInstance = axios.create({
        headers: {
            Authorization : `Bearer ${localStorage.getItem('access_token')}`
            }
      });
  
  const stopwatch = useStopwatch();
  stopwatch.reset();
  stopwatch.pause();

  export default {
    // state
    data() {
      return {
        workStatus : "offline",
        currentSeconds : 0,
        currentMinutes : 0,
        currentHours : 0,
        employeeId: "",
        is_data_fetched: false
      }
    },
    mounted () {
      reqInstance.get('https://api-test.sesametime.com/schedule/v1/work-entries')
          .then(response => {
              let lastEntry = response.data.data.sort((b, a) => a.createdAt.localeCompare(b.createdAt))[0]
              this.workStatus = lastEntry.employee.workStatus;
              let duration = moment.duration({'seconds' : lastEntry.workedSeconds});
              this.currentSeconds = duration._data.seconds;
              this.currentMinutes = duration._data.minutes;
              this.currentHours = duration._data.hours;
              this.employeeId = lastEntry.employee.id;
              this.is_data_fetched = true;
            })
    },
  }
</script>
