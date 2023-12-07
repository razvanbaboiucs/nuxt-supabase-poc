<template>
    <div class="min-h-full dark:bg-slate-900">
        <nav class="px-6 py-2.5 bg-slate-800">
            <div class="flex flex-wrap justify-between items-center mx-auto max-w-screen-xl">
                <div class="flex items-center">
                    <img src="/logo.svg" class="mr-3 h-9" alt="logo" />
                    <span class="self-center text-xl font-semibold whitespace-nowrap text-white">Nuxt + Supabase</span>
                </div>
                <UContainer>
                    <ul class="flex font-medium flex-row space-x-8 mt-0">
                        <li>
                            <ULink to="/" active-class="text-primary"
                                inactive-class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200">
                                Home
                            </ULink>
                        </li>
                        <li>
                            <ULink to="/profile" active-class="text-primary"
                                inactive-class="text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-200">
                                Profile
                            </ULink>
                        </li>
                    </ul>
                </UContainer>
                <div class="flex items-center">
                    <UButton label="Logout" color="white" variant="ghost" @click="logout" />
                </div>
            </div>
        </nav>

        <div class="py-10">
            <main>
                <div class="mx-auto max-w-7xl sm:px-6 lg:px-8 px-2">
                    <slot />
                </div>
            </main>
        </div>
    </div>
</template>

<script setup>

const supabase = useSupabaseClient()
const user = useSupabaseUser()
async function logout() {
    await supabase.auth.signOut()
}

watch(user, () => {
    if (!user.value) {
        return navigateTo('/login')
    }
}, { immediate: true })
</script>