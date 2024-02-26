<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref("");

const input_content = ref("");
const input_category = ref(null);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

const todos_asc = computed(() => {
  if (input_category.value === "important") {
    return todos.value.slice().sort((a, b) => b.createdAt - a.createdAt);
  }
  return todos.value.slice().sort((a, b) => a.createdAt - b.createdAt);
});

// Move the sort logic inside onMounted
onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  const storedTodos = JSON.parse(localStorage.getItem("todos")) || [];

  // Check if storedTodos is an array before sorting
  if (Array.isArray(storedTodos)) {
    todos.value = storedTodos.sort((a, b) => a.createdAt - b.createdAt);
  } else {
    todos.value = [];
  }
});

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
    if (Array.isArray(newVal)) {
      localStorage.setItem(
        "todos",
        JSON.stringify(newVal.sort((a, b) => a.createdAt - b.createdAt))
      );
    }
  },
  {
    deep: true,
  }
);

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    return;
  }

  const newTodo = {
    content: input_content.value,
    category: input_category.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime(),
  };

  todos.value = [...todos.value, newTodo];
  input_content.value = "";
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<template>
  <div>
    <main>
      <div class="flex justify-center">
        <section class="greeting text-slate-50">
          <h2 class="text-xl m-8">
            Name:
            <input
              type="text"
              id="name"
              placeholder="Name here"
              v-model="name"
              class="bg-transparent font-bold ms-5"
            />
          </h2>
        </section>
      </div>
      <section class="m-8">
        <h3 class="flex justify-center text-slate-50 ml my-4 text-2xl font-bold text-slate-50">CREATE A TODO</h3>
        <form id="new-todo-form" @submit.prevent="addTodo">
          <div class="grid justify-items-center">
            <div class="bg-orange-400 size-24 w-3/4 rounded-full">
              <div class="flex justify-center py-4">
                <h4 class="text-slate-50 text-2xl pr-2 py-4">
                  What's on your todo list?
                </h4>
                <input
                  type="text"
                  name="content"
                  id="content"
                  placeholder="e.g. do homework"
                  v-model="input_content"
                  class="text-lg size-16 w-3/4 p-2 border border-gray-300 rounded-md"
                />
              </div>
            </div>
          </div>
          <div class="grid justify-items-center my-8">
            <h4 class="text-slate-50 text-2xl font-bold">Pick a category</h4>
            <div class="flex justify-center my-4">
              <div class="flex items-stretch space-x-4">
                <div
                  class="bg-orange-300 size-16 w-96 p-2 border border-gray-300 rounded-md"
                >
                  <label class="flex justify-center">
                    <input
                      type="radio"
                      name="category"
                      id="category1"
                      value="important"
                      v-model="input_category"
                      class="m-4"
                    />
                    <h2 class="text-xl py-1.5">Important</h2>
                  </label>
                </div>
                <div
                  class="bg-orange-300 size-16 w-96 p-2 border border-gray-300 rounded-md"
                >
                  <label class="flex justify-center">
                    <input
                      type="radio"
                      name="category"
                      id="category2"
                      value="important"
                      v-model="input_category"
                      class="m-4"
                    />
                    <h2 class="text-xl py-1.5">Not Important</h2>
                  </label>
                </div>
              </div>
            </div>
          </div>
          <div class="grid justify-items-center">
            <div
              class="flex justify-center bg-orange-50 size-16 w-96 rounded-full"
            >
              <input
                type="submit"
                value="ADD TO THE LIST"
                class="text-xl font-bold text-slate-900"
              />
            </div>
          </div>
        </form>
      </section>

      <div class="grid justify-items-center">
        <div>
          <section>
            <h3
              class="flex justify-center text-xl font-bold text-orange-50 m-4"
            >
              TODO LIST
            </h3>

            <div class="space-y-4" id="todo-list">
              <div v-for="todo in todos_asc" :class="` ${todo.done && 'done'}`">
                <div class="flex justify-center bg-orange-300 size-25 w-96 rounded-full py-3 ">
                  <label >
                    <input type="checkbox" v-model="todo.done" class="size-5 rounded-lg me-8"/>
                    <span
                      :class="`bubble ${
                        todo.category == 'important'
                          ? 'important'
                          : 'not-important'
                      }`"
                    ></span>
                  </label>

                  <div class="">
                    <input type="text" v-model="todo.content" />
                  </div>

                  <div class="ms-8 bg-orange-50 h-9 w-12 rounded-lg">
                    <button class="delete" @click="removeTodo(todo)">
                      Delete
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>
      </div>
    </main>
  </div>
</template>
