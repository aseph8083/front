<script context="module">
  export async function preload({ params, query }) {
    return { dataParam: params };
  }
</script>

<script>
  import { onMount } from "svelte";
  import { authenticated } from "../../stores/auth";
  import { goto } from "@sapper/app.mjs";
  import {
    Button,
    Modal,
    ModalBody,
    ModalFooter,
    ModalHeader,
  } from "sveltestrap";
  let open = false;
  const toggle = () => (open = !open);
  export let dataParam;
  let name = "",
    description = "",
    token = "";

    let pic;
    let sub_organization_id;

  let dataListUser = [];
  let dataListOrganisasi = [];

  onMount(async () => {
    try {
      authenticated.subscribe(async (value) => {
        if (value) {
          // anggita
          await fetch("http://localhost:4000/api/manajemen_organisasi/user", {
            method: "GET",
            headers: {
              Accept: "application/json",
              "Content-Type": "application/x-www-form-urlencoded",
              Authorization: `${value}`,
            },
          })
            .then((response) => response.json())
            .then(async (responseJson) => {
              if (responseJson.metadata.http_code == "200") {
                dataListUser = responseJson.data;
                token = value;
              }
            })
            .catch((error) => {
              console.error(error);
              token = "";
            });

            // organisasi
            await fetch(
            "http://localhost:4000/api/manajemen_organisasi/orgnization",
            {
              method: "GET",
              headers: {
                Accept: "application/json",
                "Content-Type": "application/x-www-form-urlencoded",
                Authorization: `${value}`,
              },
            }
          )
            .then((response) => response.json())
            .then(async (responseJson) => {
              if (responseJson.metadata.http_code == "200") {
                dataListOrganisasi = responseJson.data;
                token = value;
              }
            })
            .catch((error) => {
              console.error(error);
              token = "";
            });

          await fetch(
            "http://localhost:4000/api/manajemen_organisasi/activity/" +
              dataParam.slug,
            {
              method: "GET",
              headers: {
                Accept: "application/json",
                "Content-Type": "application/x-www-form-urlencoded",
              },
            }
          )
            .then((response) => response.json())
            .then(async (responseJson) => {
              if (responseJson.metadata.http_code == "200") {
                name = responseJson.data.name;
                description = responseJson.data.description;
                pic = responseJson.data.pic;
                sub_organization_id = responseJson.data.sub_organization_id;
              }
            })
            .catch((error) => {
              console.error(error);
            });
        } else {
          window.location.href = "/login";
        }
      });
    } catch (e) {
      authenticated.set("");
      token = "";
      window.location.href = "/login";
    }
  });

  const submit = async () => {
    var details = {
      name,
      description,
      address,
    };

    var formBody = [];
    for (var property in details) {
      var encodedKey = encodeURIComponent(property);
      var encodedValue = encodeURIComponent(details[property]);
      formBody.push(encodedKey + "=" + encodedValue);
    }
    formBody = formBody.join("&");

    await fetch(
      "http://localhost:4000/api/manajemen_organisasi/activity/" +
        dataParam.slug,
      {
        method: "PUT",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/x-www-form-urlencoded",
        },
        body: formBody,
      }
    )
      .then((response) => response.json())
      .then(async (responseJson) => {
        if (responseJson.metadata.http_code == "200") {
          alert("berhasil Ditambahkan");
          await goto("/activity");
        }
      })
      .catch((error) => {
        alert("gagal");
      });
  };

  const deleteData = async (id) => {
    authenticated.subscribe(async (value) => {
      if (value) {
        await fetch(
          "http://localhost:4000/api/manajemen_organisasi/activity/" + id,
          {
            method: "DELETE",
          }
        )
          .then((response) => response.json())
          .then(async (responseJson) => {
            console.log(responseJson);
            if (responseJson.metadata.http_code == "200") {
              alert("Data berhasil dihapus");
              toggle();
              goto("/activity");
            }
          })
          .catch((error) => {
            alert("Data gagal dihapus");
            toggle();
          });
      }
    });
  };
</script>

<div class="row">
  <nav aria-label="breadcrumb">
    <ol class="breadcrumb">
      <li class="breadcrumb-item"><a href="/">Home</a></li>
      <li class="breadcrumb-item">
        <a href="/activity">Activity</a>
      </li>
      <li class="breadcrumb-item" />
      <li class="breadcrumb-item active" aria-current="page">Input Activity</li>
    </ol>
  </nav>
</div>
<div class="container">
  <form on:submit|preventDefault={submit}>
    <div class="form-group first mb-2">
      <label for="nama">Nama</label>
      <input type="text" class="form-control" id="nama" bind:value={name} />
    </div>
    <div class="form-group">
      <label for="deskripsi">Deskripsi</label>
      <input
        type="text"
        id="deskripsi"
        class="form-control"
        bind:value={description}
      />
    </div>
    <div class="form-group last mb-2">
      <label for="pic">Pic</label>
      <select bind:value={pic} class="form-select" aria-label="Default select example">
        {#each dataListUser as data}
          <option value={data.id}>{data.name}</option>
        {/each}
      </select>
    </div>

    <div class="form-group last mb-2">
      <label for="organisasi">Sub organisasi</label>
      <select bind:value={sub_organization_id}  class="form-select" aria-label="Default select example">
        <option selected>pilih sub oraganisasi</option>
        {#each dataListOrganisasi as data}
          <option value={data.id}>{data.name}</option>
        {/each}
      </select>
    </div>

    <input type="submit" value="Simpan" class="btn btn-block btn-primary" />
  </form>
  <div class="text-end margin-min-top-40">
    <Button color="danger" on:click={toggle}>Hapus</Button>
  </div>
</div>

<Modal isOpen={open} {toggle}>
  <ModalHeader {toggle}>Hapus Data</ModalHeader>
  <ModalBody>Apakah anda yakin akan menghapus data?</ModalBody>
  <ModalFooter>
    <Button color="danger" on:click={deleteData(dataParam.slug)}>Hapus</Button>
    <Button color="secondary" on:click={toggle}>Cancel</Button>
  </ModalFooter>
</Modal>
