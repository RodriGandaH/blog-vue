<template>
  <div class="container my-5">
    <h3 class="text-center">Mi Blog</h3>
    <div v-if="vista == 'list'" class="d-flex justify-content-end my-3">
      <button @click="vista = 'form'" class="btn btn-dark">Nuevo</button>
    </div>
    <div v-if="vista == 'list'" class="row justify-content-center">
      <div v-for="item in posts" :key="item.id" class="col-md-6">
        <div class="card mb-3" style="max-width: 540px">
          <div class="row g-0">
            <div class="col-md-4">
              <img
                :src="'http://hamilo.josema.site/webpage/blogs/' + item.imagen"
                class="img-fluid rounded-start"
                alt="img"
                onerror="this.src='https://via.placeholder.com/150'"
              />
            </div>
            <div class="col-md-8">
              <div class="card-body">
                <h5 class="card-title">{{ item.titulo }}</h5>
                <p class="card-text">
                  {{ item.contenido }}
                </p>

                <p class="card-text">
                  Tags:
                  <small
                    v-for="tag in item.tags != null ? item.tags.split(',') : []"
                    :key="tag"
                    class="badge bg-dark mx-1"
                    >{{ tag }}</small
                  >
                </p>

                <p class="card-text">
                  <small class="text-muted"
                    >Meta title: {{ item.meta_title }}</small
                  >
                </p>

                <p class="card-text">
                  <small class="text-muted"
                    >Meta description: {{ item.meta_description }}</small
                  >
                </p>

                <p class="card-text d-flex justify-content-between">
                  <small class="text-body-secondary">{{ item.fecha }}</small>
                  <small class="text-body-secondary">{{
                    item.usuario ? item.usuario.name : ""
                  }}</small>
                </p>
                <div class="d-flex justify-content-end">
                  <button class="btn btn-warning" @click="editar(item)">
                    Editar
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="row justify-content-center">
      <div class="col-md-7">
        <div class="card">
          <div class="card-body">
            <h5 class="card-title text-center">Registrar Post</h5>

            <div class="mb-3">
              <label for="imagen" class="form-label">Imagen</label>
              <input
                type="file"
                id="imagen"
                class="form-control"
                accept="image/*"
                @change="cargarImagen($event)"
              />

              <div class="mb-3">
                <label for="titulo" class="form-label">Titulo</label>
                <input
                  type="text"
                  v-model="titulo"
                  id="titulo"
                  class="form-control"
                />
              </div>
              <div class="mb-3">
                <label for="contenido" class="form-label">Contenido</label>
                <textarea
                  cols="30"
                  rows="10"
                  v-model="contenido"
                  id="contenido"
                  class="form-control"
                />
              </div>
              <div class="mb-3">
                <label for="tags" class="form-label"
                  >Tags <small class="text-info">(Separado por comas)</small>
                </label>
                <input
                  type="text"
                  v-model="tags"
                  id="tags"
                  class="form-control"
                />
              </div>
              <h5>Meta</h5>
              <div class="mb-3">
                <label for="meta_title" class="form-label">Meta title</label>
                <input
                  type="text"
                  v-model="meta_title"
                  id="meta_title"
                  class="form-control"
                />
              </div>
              <div class="mb-3">
                <label for="meta_description" class="form-label"
                  >Meta description</label
                >
                <input
                  type="text"
                  v-model="meta_description"
                  id="meta_description"
                  class="form-control"
                />
              </div>
              <div class="text-center my-3">
                <button
                  v-if="!seleccionado.id"
                  class="btn btn-dark mx-2"
                  @click="registrar()"
                >
                  Registrar Post
                </button>

                <button v-else class="btn btn-dark mx-2" @click="actualizar()">
                  Actualizar Post
                </button>
                <button @click="vista = 'list'" class="btn btn-danger mx-2">
                  Cancelar
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

const apiUrl = "http://hamilo.josema.site/api/v1/";

const usuario = "rodrigo.gandarillas.heredia@gmail.com";
const password = "rodri1704";

const vista = ref("list");

const posts = ref([]);
const imagen = ref(null);
const seleccionado = ref({});

const titulo = ref("");
const contenido = ref("");
const tags = ref("");
const meta_title = ref("");
const meta_description = ref("");

onMounted(() => {
  cargarDatos();
});

const limpiarCampos = () => {
  titulo.value = "";
  contenido.value = "";
  tags.value = "";
  meta_title.value = "";
  meta_description.value = "";
  imagen.value = null;
};

const cargarDatos = async () => {
  try {
    const response = await axios.get(apiUrl + "posts", {
      headers: {
        Authorization: "Basic " + btoa(usuario + ":" + password),
      },
    });
    posts.value = response.data;
    console.log(response);
  } catch (error) {
    console.log(error);
  }
};
const cargarImagen = (event) => {
  imagen.value = event.target.files[0];
  console.log(imagen.value);
};

const registrar = async () => {
  if (
    titulo.value == "" ||
    contenido.value == "" ||
    tags.value == "" ||
    meta_title.value == "" ||
    meta_description.value == ""
  ) {
    alert("Todos los campos son obligatorios");
    return;
  }

  try {
    const post = {
      titulo: titulo.value,
      contenido: contenido.value,
      tags: tags.value,
      meta_title: meta_title.value,
      meta_description: meta_description.value,
      estado: 1,
      imagen: imagen.value,
    };

    const response = await axios.post(apiUrl + "post", post, {
      headers: {
        Authorization: "Basic " + btoa(usuario + ":" + password),
        "Content-Type": "multipart/form-data",
        //ContentType: "application/json",
        Accept: "application/json",
      },
    });
    console.log(response);
    if (response.status == 201) {
      cargarDatos();
      limpiarCampos();
      vista.value = "list";
    }
  } catch (error) {
    console.log(error);
  }
};
const editar = (item) => {
  seleccionado.value = item;

  titulo.value = item.titulo;
  contenido.value = item.contenido;
  tags.value = item.tags;
  meta_title.value = item.meta_title;
  meta_description.value = item.meta_description;

  vista.value = "form";
};

const actualizar = async () => {
  if (
    titulo.value == "" ||
    contenido.value == "" ||
    tags.value == "" ||
    meta_title.value == "" ||
    meta_description.value == ""
  ) {
    alert("Todos los campos son obligatorios");
    return;
  }

  try {
    const post = {
      titulo: titulo.value,
      contenido: contenido.value,
      tags: tags.value,
      meta_title: meta_title.value,
      meta_description: meta_description.value,
      estado: 1,
      imagen: imagen.value,
      _method: "PUT",
    };

    const response = await axios.post(
      apiUrl + "post/" + seleccionado.value.id,
      post,
      {
        headers: {
          Authorization: "Basic " + btoa(usuario + ":" + password),
          "Content-Type": "multipart/form-data",
          //ContentType: "application/json",
          Accept: "application/json",
        },
      }
    );
    console.log(response);
    console.log(response);
    if (response.status == 201) {
      cargarDatos();
      limpiarCampos();
      vista.value = "list";
    }
  } catch (error) {
    console.log(error);
  }
};
</script>
<style scoped>
</style>
