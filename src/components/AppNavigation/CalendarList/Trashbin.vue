<!--
  - @copyright 2021 Christoph Wurst <christoph@winzerhof-wurst.at>
  -
  - @author 2021 Christoph Wurst <christoph@winzerhof-wurst.at>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<template>
	<AppNavigationItem :title="t('calendar', 'Trashbin')"
										 :pinned="true"
										 @click.prevent="showModal = true">
		<template #extra>
			<Modal v-if="showModal"
						 @close="showModal = false">
				<div class="modal__content">
					<h2>{{ t('calendar', 'Trashbin') }}</h2>
					<table>
						<tr>
							<th>{{ t('calendar', 'Name') }}</th>
							<th>{{ t('calendar', 'Deleted at') }}</th>
							<th>&nbsp;</th>
						</tr>
						<tr v-for="item in items">
							<td>{{ item.name }}</td>
							<td>{{ item.deletedAt }}</td>
							<td><button @click="restore(item)">restore</button></td>
						</tr>
					</table>
				</div>
			</Modal>
		</template>
	</AppNavigationItem>
</template>

<script>
import AppNavigationItem from '@nextcloud/vue/dist/Components/AppNavigationItem'
import AppNavigationCounter from '@nextcloud/vue/dist/Components/AppNavigationCounter'
import Modal from '@nextcloud/vue/dist/Components/Modal'
import logger from '../../../utils/logger'
import {getCalendarHomeUrl, move} from "../../../services/caldavService";
import {showError} from "@nextcloud/dialogs";

export default {
	name: 'Trashbin',
	components: {
		AppNavigationCounter,
		AppNavigationItem,
		Modal,
	},
	data() {
		return {
			showModal: false,
		}
	},
	computed: {
		calendars() {
			return this.$store.getters.sortedDeletedCalendars
		},
		objects() {
			return []
		},
		items() {
			const formattedCalendars = this.calendars.map(calendar => ({
				calendar,
				type: 'calendar',
				key: calendar.url,
				name: calendar.displayname,
				url: calendar._url,
				deletedAt: calendar._props['{http://nextcloud.com/ns}deleted-at']
			}))
			const formattedCalendarObjects = this.objects.map(object => ({
				type: 'object',
				key: object.id,
				name: object.summary,
			}))

			return formattedCalendars.concat(formattedCalendarObjects)
		},
	},
	methods: {
		async restore(item) {
			logger.debug('restoring ' + item.url, item)
			try {
				await move(item.url, getCalendarHomeUrl() + '/trashbin/restore/xyz')

				switch(item.type) {
					case 'calendar':
						this.$store.dispatch('removeRestoredCalendar', { calendar: item.calendar })
						this.$store.dispatch('loadCalendars')
						break;
					case 'object':

						break;
				}
			} catch (error) {
				logger.error('could not restore ' + item.url, error)

				showError(t('calendar', 'Could not restore calendar or event'))
			}
		}
	}
}
</script>

<style scoped>
.modal__content {
	width: 40vw;
	margin: 2vw;
}
</style>
