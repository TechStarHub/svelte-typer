<script>
    import Header from "../components/Header.svelte";
    import Footer from "../components/Footer.svelte";
</script>

<main class="w-full min-h-screen flex flex-col justify-between items-center bg-[#9EB8D9] ">
    <Header />
    <slot/>
    <Footer />
</main>