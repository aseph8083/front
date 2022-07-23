<script>
  import { onMount } from "svelte";
  import { authenticated } from "../../stores/auth";
  import { goto } from "@sapper/app.mjs";
  let name = "",
    description = "",
    address = "",
    token = "";

    let pic;
    let sub_organization_id;

  let dataListUser = [];
  let dataListOrganisasi = [];

  onMount(async () => {
    try {
      authenticated.subscribe(async (value) => {
        if (value) {
          token = value;
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
    pic,
    sub_organization_id,
    };

    console.log(details);

    var formBody = [];
    for (var property in details) {
      var encodedKey = encodeURIComponent(property);
      var encodedValue = encodeURIComponent(details[property]);
      formBody.push(encodedKey + "=" + encodedValue);
    }
    formBody = formBody.join("&");

    await fetch("http://localhost:4000/api/manajemen_organisasi/activity", {
      method: "POST",
      headers: {
        Accept: "application/json",
        "Content-Type": "application/x-www-form-urlencoded",
      },
      body: formBody,
    })
      .then((response) => response.json())
      .then(async (responseJson) => {
        if (responseJson.metadata.http_code == "201") {
          alert("berhasil Ditambahkan");
          goto("/activity");
        }
      })
      .catch((error) => {
        alert("gagal");
      });
  };
</script>

{#if token != ""}
  <div class="row">
    <nav aria-label="breadcrumb">
      <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="/">Home</a></li>
        <li class="breadcrumb-item">
          <a href="/activity">Activity</a>
        </li>
        <li class="breadcrumb-item" />
        <li class="breadcrumb-item active" aria-current="page">
          Input Activity
        </li>
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
  </div>
{/if}
