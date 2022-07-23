<script>
  import { onMount } from "svelte";
  import { authenticated } from "../../stores/auth";
  import { goto } from "@sapper/app.mjs";
  import { Button } from "sveltestrap";

  let token = "";
  let dataList = [];
  onMount(async () => {
    try {
      authenticated.subscribe(async (value) => {
        if (value) {
          await fetch(
            "http://localhost:4000/api/manajemen_organisasi/activity",
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
                dataList = responseJson.data;
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

  const editData = (id) => {
    console.log(id);
    goto(`/activity/${id}`);
  };
</script>

{#if token != ""}
  <div class="container-fluid">
    <div class="row">
      <div class="col">
        <nav aria-label="breadcrumb">
          <ol class="breadcrumb">
            <li class="breadcrumb-item"><a href="/">Home</a></li>
            <li class="breadcrumb-item">
              <a href="/activity">Activity</a>
            </li>
          </ol>
        </nav>
      </div>
      <div class="col">
        <div class="text-end">
          <a href="/activity/input">
            <button type="button" class="btn btn-primary">Tambah</button>
          </a>
        </div>
      </div>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">Nama</th>
          <th scope="col">Deskripsi</th>
          <th scope="col">Alamat</th>
          <th scope="col">Action</th>
        </tr>
      </thead>
      <tbody>
        {#each dataList as data}
          <tr>
            <td>{data.name != undefined ? data.name : "-"}</td>
            <td>{data.description != undefined ? data.description : "-"}</td>
            <td>{data.address != undefined ? data.address : "-"}</td
            >
            <td
              ><Button color="primary" on:click={editData(data.id)}>Edit</Button
              ></td
            >
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
{/if}
