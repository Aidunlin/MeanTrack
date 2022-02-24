<script lang="ts" context="module">
  import type {
    FirestoreDataConverter,
    Timestamp,
  } from "firebase/firestore";

  export interface UserData {
    id: string;
    hours: number;
    lastAction: Timestamp;
    name: string;
    teamId: string;
    tracking: boolean;
  }

  export interface TeamData {
    id: string;
    members: string[];
    name: string;
    number: number;
    ownerId: string;
  }

  export const userDataConv: FirestoreDataConverter<UserData> = {
    toFirestore: (data: UserData) => {
      return {
        hours: data.hours,
        lastAction: data.lastAction,
        name: data.name,
        teamId: data.teamId,
        tracking: data.tracking,
      };
    },
    fromFirestore: (snapshot, options) => {
      const user = snapshot.data(options);
      return {
        id: snapshot.id,
        hours: user.hours,
        lastAction: user.lastAction,
        name: user.name,
        teamId: user.teamId,
        tracking: user.tracking,
      };
    },
  };

  export const teamDataConv: FirestoreDataConverter<TeamData> = {
    toFirestore: (data: TeamData) => {
      return {
        members: data.members,
        name: data.name,
        number: data.number,
        ownerId: data.ownerId,
      };
    },
    fromFirestore: (snapshot, options) => {
      const team = snapshot.data(options);
      return {
        id: snapshot.id,
        members: team.members,
        name: team.name,
        number: team.number,
        ownerId: team.ownerId,
      };
    },
  };
</script>
