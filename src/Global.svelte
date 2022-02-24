<script lang="ts" context="module">
  import type { Auth } from "firebase/auth";
  import type {
    CollectionReference,
    DocumentReference,
    FirestoreDataConverter,
    Timestamp,
  } from "firebase/firestore";
  import { writable } from "svelte/store";

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
    goal: number;
    name: string;
    number: number;
    ownerId: string;
  }

  export interface MT {
    auth: Auth;
    userData: UserData;
    userDocument: DocumentReference<UserData>;
    userCollection: CollectionReference<UserData>;
    teamData: TeamData;
    teamDocument: DocumentReference<TeamData>;
    teamCollection: CollectionReference<TeamData>;
  }

  export const mt = writable<MT>({
    auth: null,
    userData: null,
    userDocument: null,
    userCollection: null,
    teamData: null,
    teamDocument: null,
    teamCollection: null,
  });

  export const convertUserData: FirestoreDataConverter<UserData> = {
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

  export const convertTeamData: FirestoreDataConverter<TeamData> = {
    toFirestore: (data: TeamData) => {
      return {
        goal: data.goal,
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
        goal: team.goal,
        members: team.members,
        name: team.name,
        number: team.number,
        ownerId: team.ownerId,
      };
    },
  };
</script>
