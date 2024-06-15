<script lang="ts">
  import Logo from '$lib/assets/logo.png';
  import Mod from '$lib/components/Mod.svelte';
  import axios from "axios";
  import {onMount} from "svelte";
  import {open} from "@tauri-apps/api/dialog";
  import {homeDir} from "@tauri-apps/api/path"
  import {readDir, type FileEntry, exists, createDir, removeFile} from "@tauri-apps/api/fs";

  import { invoke } from "@tauri-apps/api/tauri";

  invoke('extract', {path: 'test'})

  let versions: string[]  = [];
  let contents: FileEntry[] = []

  onMount(async () => {
    let prevContents: FileEntry[] = [];
    try {
      let response = await axios.get("https://launchermeta.mojang.com/mc/game/version_manifest.json");
      versions = response.data.versions.filter((version: {type: string}) => version.type==="release").map((version: {id: string}) => version.id)
    
      // const path = await homeDir() + "AppData\\Roaming\\.minecraft\\modater";
      // if (!(await exists(path))) {
      //   await createDir(path);
      // } else {
      //   prevContents = await readDir(path);
      //   for (const file of prevContents) {
      //     await removeFile(file.path);
      //   }
      // }
    
    } catch(e) {
      console.log(e);
    }
  })

  const openHandler = async () => {
    try {
      const homeDirPath = await homeDir();
      const selectedPath = await open({
        multiple: false,
        title: "Open Mod Folder",
        directory: true,
        defaultPath: homeDirPath + "/AppData/Roaming/.minecraft"
      });
      contents = await readDir(selectedPath as string);
    } catch (e) {
      console.log(e);
    }
  }
</script>

<div class="h-[600px] w-[800] bg-zinc-900 flex flex-col justify-center items-center">
  <img src="{Logo}" alt="" class="max-w-[200px] max-h-[50px]">
  <div class="overflow-auto w-[600px] h-[400px] rounded-3xl border-solid border-zinc-800 border-2 m-6">
    <ul>
      {#each contents as content, i}
         <Mod fileName={content.name} modName="Test" bgColor={`${i%2===0? "bg-zinc-700" : "bg-zinc-800"}`}/>
      {/each}
    </ul>
  </div>
  <div class="space-x-3 flex">
    <button on:click={openHandler} class="text-white bg-emerald-500 rounded-2xl flex justify-center items-center pr-4 pl-4 pt-1 pb-1">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="mr-2" viewBox="0 0 16 16">
        <path d="M9.828 3h3.982a2 2 0 0 1 1.992 2.181l-.637 7A2 2 0 0 1 13.174 14H2.825a2 2 0 0 1-1.991-1.819l-.637-7a2 2 0 0 1 .342-1.31L.5 3a2 2 0 0 1 2-2h3.672a2 2 0 0 1 1.414.586l.828.828A2 2 0 0 0 9.828 3m-8.322.12q.322-.119.684-.12h5.396l-.707-.707A1 1 0 0 0 6.172 2H2.5a1 1 0 0 0-1 .981z"/>
      </svg>
      open
    </button>
    <select class="rounded-2xl pr-4 pl-4">
      {#each versions as version}
        <option value="">{version}</option>
      {/each}
    </select>
    <button class="text-white bg-emerald-500 rounded-2xl flex justify-center items-center pr-4 pl-4 pt-1 pb-1">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="mr-2" viewBox="0 0 16 16">
        <path d="M16 8A8 8 0 1 0 0 8a8 8 0 0 0 16 0m-7.5 3.5a.5.5 0 0 1-1 0V5.707L5.354 7.854a.5.5 0 1 1-.708-.708l3-3a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 5.707z"/>
      </svg>
      update
    </button>
  </div>
</div>