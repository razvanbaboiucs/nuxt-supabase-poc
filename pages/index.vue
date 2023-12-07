<template>
    <UBadge class="mb-4 text-sm" variant="outline">
        {{ role === 'admin' ? 'Admin' : 'Basic user' }}
    </UBadge>
    <div v-if="role === 'admin'">
        <AdminView />
    </div>
    <div v-else>
        <BasicUserView />
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()
const user = useSupabaseUser()
const { data: role } = await useAsyncData('role', getRole)

async function getRole() {
    const { data: roles, error } = await supabase.from('users').select('role').eq('id', user.value.id)
    if (error) {
        console.error(error)
        return null
    }
    return roles[0].role
}
</script>