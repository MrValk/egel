<script lang="ts">
	import Fa from 'svelte-fa';
	import { faCheckCircle, faClock } from '@fortawesome/free-regular-svg-icons';
	import { faGamepad, faPaintBrush, faPerson } from '@fortawesome/free-solid-svg-icons';
	import { Circle } from 'svelte-loading-spinners/src';

	import { page } from '$app/stores';
	import type { UserDB } from 'src/models/user';
	import { onMount } from 'svelte';

	import { getUser } from '../../../../database/users';
	import { formatDate } from '../../../../scripts/formatDate';
	import { authStore } from '../../../../stores/auth';
	import { reloadPosts } from '../../../../stores/posts';
	import { postsStore } from '../../../../stores/posts';

	import LinkWrapper from '../../../../components/general/linkWrapper.svelte';
	import LgCard from '../../../../components/general/cards/lgCard.svelte';
	import PfPic from '../../../../components/user/pfPic.svelte';
	import Line from '../../../../components/general/line.svelte';
	import WideLine from '../../../../components/general/wideLine.svelte';
	import Gallery from '../../../../components/user/gallery.svelte';
	import NewPost from '../../../../components/user/newPost.svelte';
	import LinksList from '../../../../components/user/linksList.svelte';

	let user: UserDB;
	let myProfile = false;

	let loaded = false;
	let loading = false;

	onMount(load);
	$: $page.params.uid, load();
	$: $authStore, load();

	async function load() {
		if (loading) return;
		loading = true;

		const temp = await getUser($page.params.uid);

		if (!temp) {
			loaded = true;
			return;
		}

		user = temp;

		myProfile = $authStore.user != null && $page.params.uid == $authStore.user.uid;

		reloadPosts($page.params.uid);

		loaded = true;
		loading = false;
	}
</script>

<LgCard>
	{#if user}
		<div class="wrapper flex flex-col gap-12 items-center">
			<div class="flex w-full gap-8 justify-between">
				<div class="flex flex-col w-2/5">
					<div
						class="flex flex-col py-8 gap-4 items-center justify-center bg-gray-200/50 rounded-lg"
					>
						<PfPic pfPicUrl={user.pfPic} />
						<h2 class="flex items-center gap-2 text-4xl mb-2">
							{#if user.accepted}
								<div class="accepted relative flex items-center justify-center rounded-full">
									<Fa icon={faCheckCircle} class="text-purple-400" />
								</div>
							{/if}
							<b>{user.username}</b>
						</h2>

						<LinksList links={user.links} />
					</div>
				</div>
				<div class="flex flex-col w-7/12 pt-6">
					{#if user.bio}
						<div>
							<h3 class="font-semibold">Over mij:</h3>
							<LinkWrapper text={user.bio} />
						</div>
					{/if}
					{#if user.bio && user.groups}
						<WideLine color="bg-gray-200" />
					{/if}
					{#if user.groups}
						<div>
							<h3 class="font-semibold">Lid van groepen:</h3>
							<div class="grid grid-cols-3">
								{#each user.groups as group}
									<p>{group}</p>
								{/each}
							</div>
						</div>
					{/if}
				</div>
			</div>
			{#if user.other}
				<div class="flex justify-center gap-24 w-full">
					{#if user.other.memberSince}
						<div class="flex flex-col items-center gap-1">
							<h5 class="flex items-center text-xl font-semibold gap-2">
								<Fa icon={faClock} />Egelaar sinds:
							</h5>
							<p class="text-lg">{formatDate(user.other?.memberSince)}</p>
						</div>
					{/if}
					{#if user.other.realName}
						<div class="flex flex-col items-center gap-1">
							<h5 class="flex items-center text-xl font-semibold gap-2">
								<Fa icon={faPerson} />Echte naam:
							</h5>
							<p class="text-lg">{user.other.realName}</p>
						</div>
					{/if}
					{#if user.other.favoriteGame}
						<div class="flex flex-col items-center gap-1">
							<h5 class="flex items-center text-xl font-semibold gap-2">
								<Fa icon={faGamepad} />Favoriete game:
							</h5>
							<p class="text-lg">{user.other.favoriteGame}</p>
						</div>
					{/if}
					{#if user.other.hobby}
						<div class="flex flex-col items-center gap-1">
							<h5 class="flex items-center text-xl font-semibold gap-2">
								<Fa icon={faPaintBrush} />Hobby:
							</h5>
							<p class="text-lg">{user.other.hobby}</p>
						</div>
					{/if}
				</div>
			{/if}
			<Line color="bg-gray-200" />
			<div class="w-full flex flex-col items-center gap-6 -mt-6">
				<div class="flex flex-col items-center w-full gap-2">
					<h2 class="text-2xl font-semibold">
						{myProfile ? 'Jouw posts' : `Posts van ${user.username}`}
					</h2>
					{#if myProfile}
						<NewPost />
					{:else if !$postsStore.length}
						<p class="text-xl mt-8">Er is hier nog niks gepost...</p>
					{/if}
				</div>
				<Gallery />
			</div>
		</div>
	{:else if loaded}
		<div class="wrapper flex flex-col items-center justify-center gap-6">
			<h2 class="text-4xl font-semibold">Egelaar niet gevonden</h2>
			<p>Deze egelaar bestaat niet of is verwijderd.</p>
		</div>
	{:else}
		<div class="wrapper flex flex-col items-center justify-center">
			<Circle color="#9CA3AF" />
		</div>
	{/if}
</LgCard>

<style lang="scss">
	.wrapper {
		min-height: 31rem;

		.accepted {
			&::after {
				content: 'Erkend Egel lid';
				position: absolute;
				display: flex;
				justify-content: center;
				top: -3.25rem;
				font-size: 1rem;
				white-space: nowrap;
				border-radius: 0.5rem;
				width: 8rem;
				background-color: rgb(209 213 219);
				opacity: 0;
				transition: opacity 150ms ease;
			}

			&:hover::after {
				opacity: 1;
			}
		}
	}
</style>
