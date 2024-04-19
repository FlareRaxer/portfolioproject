<script setup>
import { ref, onMounted, onBeforeUnmount, computed} from 'vue';

const title = ref('');
const article = ref('');
const posts = ref([]); // Array to store fetched posts
const image = ref(null); // Ref for the uploaded image
const selectedPost = ref(null);
const currentPage = ref(1);
const postsPerPage = ref(6);
const errors = ref({});

const onSubmitform = () => {
  validateForm();

  if (Object.keys(errors.value).length > 0) {
    return;
  }

  fetch('https://portfolio-project-99753-default-rtdb.europe-west1.firebasedatabase.app/portfolio.json', {
    method: 'POST',
    body: JSON.stringify({
      title: title.value, // Assign the value of title.value to the title property
      article: article.value,
      image: image.value, // Assuming you have logic to get the image URL
    })
  })
  .then(response => response.json())
  .then(data => {
    // Fetch the updated list of posts
    fetch('https://portfolio-project-99753-default-rtdb.europe-west1.firebasedatabase.app/portfolio.json')
      .then(response => response.json())
      .then(data => {
        posts.value = Object.values(data); // Update the posts array
        currentPage.value = 1; // Go to the first page to see the new post
      });
    // Clear the input fields
    title.value = '';
    article.value = '';
    image.value = null;
  })
}



const validateForm = () => {
  errors.value = {};

  if (!title.value) {
    errors.value.title = 'Title is required';
  }

  if (!article.value) {
    errors.value.article = 'Article is required';
  }
};


const onFileChange = (e) => {
  let files = e.target.files || e.dataTransfer.files;
  if (!files.length)
    return;
  createImage(files[0]);
}

const createImage = (file) => {
    // Check if the file is an image
    if (!file.type.startsWith('image/')) {
    errors.value.image = 'File is not an image';
    return;
  }
  let reader = new FileReader();
  reader.onload = (e) => {
    image.value = e.target.result;
  };
  reader.readAsDataURL(file);
}

// Function to auto expand textarea
const autoExpand = (element) => {
  element.style.height = 'auto';
  element.style.height = element.scrollHeight + 'px';
}

onMounted(() => {
  const textarea = document.querySelector('.textarea-field');
  textarea.addEventListener('input', () => autoExpand(textarea));
});

onBeforeUnmount(() => {
  const textarea = document.querySelector('.textarea-field');
  textarea.removeEventListener('input', () => autoExpand(textarea));
});

// Fetch posts on component mount
onMounted(async () => {
  const response = await fetch('https://portfolio-project-99753-default-rtdb.europe-west1.firebasedatabase.app/portfolio.json');
  const data = await response.json();
  // Convert the object to an array
  posts.value = Object.keys(data).map(key => ({id: key, ...data[key]}));
})

const paginatedPosts = computed(() => {
  const start = Math.max(0, posts.value.length - currentPage.value * postsPerPage.value);
  const end = Math.min(posts.value.length, start + postsPerPage.value);
  return posts.value.slice(start, end).reverse(); // Reverse the array to show latest posts first
});

const deletePost = (postId) => {
  fetch(`https://portfolio-project-99753-default-rtdb.europe-west1.firebasedatabase.app/portfolio/${postId}.json`, {
    method: 'DELETE'
  })
  .then(() => {
    // Remove the post from the posts array
    const index = posts.value.findIndex(post => post.id === postId);
    if (index !== -1) {
      posts.value.splice(index, 1);
    }
  });
};

</script>


<template>
  <div class="main-container">
    <div class="container">
      <form v-on:submit.prevent="onSubmitform">
        <h2 class="rubik-600">Indtast navnet på artikel</h2>
        <input type="text" v-model="title" />
        <p v-if="errors.title" class="error">{{ errors.title }}</p>
        <h2 class="rubik-600">Skriv artikel</h2>
        <textarea v-model="article" class="textarea-field" rows="10"></textarea>
        <p v-if="errors.article" class="error">{{ errors.article }}</p>
        <h2 class="rubik-600">Indsæt billede</h2>
        <input type="file" @change="onFileChange" />
        <h2 class="rubik-600"> </h2>
        <button type="submit">Post artikel</button>
        <h2 class="rubik-600"> </h2>
      </form>
    </div>

    <div class="display-container">
      <div class="delete-container" v-for="post in paginatedPosts" :key="post.id">
        <h2 class="rubik-600" @click="selectedPost = post">
        {{ post.title }}
        </h2>
        <button @click.stop="deletePost(post.id)">Delete</button>
      </div>
        <br>
      <div class="button-container">
        <button @click="currentPage++" :disabled="currentPage * postsPerPage >= posts.length">Next</button> 
        <button @click="currentPage--" :disabled="currentPage === 1">Back</button>
      </div>
    </div>  
  </div>
  

  <div v-if="selectedPost" class="selected-post-container">
    <h2 class="rubik-600">{{ selectedPost.title }}</h2>
    <p>{{ selectedPost.article }}</p>
    <img v-if="selectedPost.image" :src="selectedPost.image" alt="Post image" />
  </div>

  <RouterView />
</template>


