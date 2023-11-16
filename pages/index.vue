<template>
    <div v-if="role === 'admin'">
        <AdminView/>
    </div>
    <div v-else>
        <BasicUserView/>
    </div>
</template>

<script setup>
const supabase = useSupabaseClient()
const {data: role} = await useAsyncData('role', getRole)

async function getRole() {
    const {data: roles, error} = await supabase.from('user_roles').select('role').limit(1)
    if (error) {
        console.error(error)
        return null
    }
    return roles[0].role
}
</script>