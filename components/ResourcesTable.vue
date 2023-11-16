<template>
  <div>
    <p class="font-bold text-primary text-2xl mb-4">Resources</p>
    <UTable :rows='rows' :columns="columns">
      <template #actions-data="{ row }">
        <UDropdown :items="actions(row)">
          <UButton color="gray" variant="ghost" icon="i-heroicons-ellipsis-horizontal-20-solid" />
        </UDropdown>
      </template>
    </UTable>
  </div>
</template>

<script setup>
const supabase = useSupabaseClient()
const { data: rows } = await useAsyncData('resources', getResources)

const columns = [{
  key: 'id',
  label: 'ID'
}, {
  key: 'name',
  label: 'Name'
}, {
  key: 'description',
  label: 'Description'
}, {
  key: 'path',
  label: 'File path'
}, {
  key: 'actions'
}]

const actions = (row) => [
  [{
    label: 'Download',
    icon: 'i-heroicons-arrow-down-tray-20-solid',
    click: () => downloadFile(row.path)
  }]
]


async function getResources() {
  const { data: resources } = await supabase.from('resources').select('id,name,description,path')
  return resources
}

async function downloadFile(path) {
  const { data } = await supabase.storage.from('resources').createSignedUrls([`public/${path}`], 60)
  window.open(data[0].signedUrl, '_blank')
}
</script>


