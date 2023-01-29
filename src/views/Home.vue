<template>
    <AddTask v-show="showAddTask" @add-task="addTask" />
    
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks"/>
</template>

<script>
    import Tasks from '../components/Tasks.vue'
    import AddTask from '../components/AddTask.vue'

    export default {
        name: 'Home',
        props: {
            showAddTask: Boolean,
        },
        components: {
            Tasks,
            AddTask
        },
        data() {
            return {
                tasks: [],
            }
        },
        methods: {
            async deleteTask(id) {
                if(confirm('Are you sture?')) {
                    const res = await fetch(`api/tasks/${id}`, {
                    method: 'DELETE'
                    })

                    res.status === 200 ? (this.tasks = this.tasks.filter((task) => task.id !== id)) : alert('Error deleting this task')
                
            }
            },
            async toggleReminder(id) {
                const taskToToggle = await this.fetchTask(id)
                console.log(taskToToggle)
                const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder}
                console.log(updateTask)
                console.log(JSON.stringify(updateTask))

                const res = await fetch(`api/tasks/${id}`, {
                    method: 'PUT',
                    headers: {
                    'Content-type': 'application/json'
                    },
                    body: JSON.stringify(updateTask)
                })

                const data = await res.json()

                this.tasks = this.tasks.map((task) =>
                task.id === id ? {...task, reminder: data.reminder} : task)
            },
            async addTask(task) {
                const res = await fetch('api/tasks', {
                    method: 'POST', 
                    headers: {
                    'Content-type': 'application/json',
                    },
                    body: JSON.stringify(task),
                })

                const data = await res.json()

                console.log(data)

                this.tasks = [...this.tasks, data]
            },
            async toggleAddTask() {
                this.showAddTask = !this.showAddTask
            },
            async fetchTasks() {
                const res = await fetch('api/tasks')
                const data = await res.json()
                return data
            },
            async fetchTask(id) {
                const res = await fetch(`api/tasks/${id}`)
                const data = await res.json()
                return data
            }
        },
        async created() {
            this.tasks = await this.fetchTasks()
        },
    }
</script>