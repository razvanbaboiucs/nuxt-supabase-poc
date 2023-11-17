<template>
    <p class="mb-5 text-2xl text-primary font-bold">Resources</p>
    <ul>
      <li v-for="resource in resourceDataRef">
        <UCard class="mb-4 max-w-3xl">
          <template #header>
            <p class="text-lg text-primary font-semibold">{{ resource.name }}</p>
          </template>

          <p>{{ resource.description }}</p>

          <template #footer>
            <div v-if="resource.status === 'accepted'">
              <UButton label="Download" @click="() => downloadFile(resource.path)"/>
            </div>
            <div v-else-if="resource.status === 'pending'">
              <UBadge label="Pending" color="cyan" variant="subtle" />
            </div>
            <div v-else-if="resource.status === 'denied'">
              <UBadge label="Access denied" color="red" variant="subtle" />
            </div>
            <div v-else>
              <UButton label="Request access" @click="() => requestAccess(resource.id)" />
            </div>
          </template>
        </UCard>
      </li>
    </ul>
</template>

<script setup>

const supabase = useSupabaseClient()
const user = useSupabaseUser()
const toast = useToast()

const { data: resourceData } = await useAsyncData('resources', getResources)
const resourceDataRef = ref(resourceData)

supabase.channel("access_requests_channel")
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'access_requests' }, payload => {
    const newResource = payload.new
    resourceDataRef.value = resourceDataRef.value.map(element => {
      if (element.id === newResource.resource_id) {
        element.status = newResource.status
        toast.add({title: 'Access to resource has been changed'})
      }
      return element
    });
  })
  .subscribe()

async function getResources() {
  function mapResource(resource) {
    return {
      id: resource.id,
      name: resource.name,
      path: resource.path,
      description: resource.description,
      status: resource.access_requests[0]?.status
    }
  }
  const { data: resources } = await supabase.from('resources').select('id,name,path,description,access_requests(status)').eq('access_requests.user_id', user.value.id)
  return resources.map(mapResource)
}

async function requestAccess(resourceId) {
  const { error } = await supabase.from('access_requests').insert({
    resource_id: resourceId,
    user_id: user.value.id,
  })
  if (error) {
    console.error(error)
  }
  else {
    resourceDataRef.value = resourceDataRef.value.map(element => {
      if (element.id === resourceId) {
        element.status = 'pending'
      }
      return element
    });
  }
}

async function downloadFile(path) {
  const { data } = await supabase.storage.from('resources').createSignedUrls([`public/${path}`], 60)
  window.open(data[0].signedUrl, '_blank')
}

</script>

