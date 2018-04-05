# Live coding GraphQL

## Prérequis

lancer le container Docker PostgreSQL si non présent
````bash
sudo docker run --name postgres-graphql -p 5432:5432 -e POSTGRES_PASSWORD=postgres-graphql -d postgres
````

télécharger DBeaver pour pouvoir accéder à la base de données: https://dbeaver.jkiss.org/download/

S'assurer que les objets précédents sont bien détruits
````sql
drop function check_may_holidays;
drop table holidays;
drop table users;
````

## Créer et peupler les objets de la base de données

````sql
create table users (
	id_dgasi text not null,
	first_name text not null,
	last_name text not null,
	primary key(id_dgasi)
);

create table holidays (
	id serial,
	start_date timestamp not null,
	end_date timestamp,
	id_dgasi text references users (id_dgasi),
	primary key(id)
);

insert into users(id_dgasi, first_name, last_name) values('ijma1234', 'Jean-Baptiste', 'Martin');
insert into users(id_dgasi, first_name, last_name) values('ivle1234', 'Valentin', 'Lefort');
insert into users(id_dgasi, first_name, last_name) values('iral1234', 'Richard', 'Albenque');
insert into users(id_dgasi, first_name, last_name) values('ijba1234', 'Jean-Philippe', 'Baconnais');
insert into users(id_dgasi, first_name, last_name) values('ejah1234', 'Jérôme', 'Ah Leung');
insert into users(id_dgasi, first_name, last_name) values('imba1234', 'Marietta', 'Baubineau');
insert into users(id_dgasi, first_name, last_name) values('istr1234', 'Sophie', 'Tremant');
insert into users(id_dgasi, first_name, last_name) values('ilha1234', 'Laurent', 'Haillot');
````

## Installer et lancer PostGraphile

NB: la version de node doit être supérieure ou égale à la 8.6.0

* installer postgraphile avec la commande suivante: `npm install -g postgraphile`

* le lanncer avec la commande suivante: `postgraphile -c postgres://postgres:postgres-graphql@10.200.164.203:5432/postgres --watch --classic-ids`

* vérifier que les points d'accès fonctionnent correctement
    * environnement de développement : http://localhost:5000/graphiql
    * point d'accès REST : http://localhost:5000/graphql

## Exécution de requêtes sur l'environnement de développement

### Les Query (interrogation de données)

accès au nom et prénom de l'utilisateur ijma1234

````graphql
query {
  userByIdDgasi(idDgasi: "ijma1234") {
    idDgasi
    firstName
    lastName
  }
}
````

accès à tous les utilisateurs de la base

````graphql
query {
  allUsers {
    edges {
      node {
        idDgasi,
        firstName,
        lastName
      }
    }
  }
}
````

accès à toutes les valeurs enregistrées

````graphql
query {
  allHolidays {
    edges {
      node {
        startDate
        endDate
      }
    }
  }
}
````

accès à toutes les vacances d'un utilisateurs

````graphql
query {
  userByIdDgasi(idDgasi: "ijma1234") {
    idDgasi
    firstName
    lastName
    holidaysByIdDgasi {
      edges {
        node {
		  id
          startDate
          endDate
        }
      }
    }
  }
}
````

### Les Mutations (mises à jour de données)

créer une nouvelle période de vacances

````graphql
mutation {
  createHoliday(input: {
    holiday: {
      startDate: "2018-03-11",
      endDate: "2018-03-11",
      idDgasi: "ijma1234"
    }
  }) {
    clientMutationId
  }
}
````

mettre à jour une période de vacances

````graphql
mutation {
  updateHoliday(input: {
    id: "WyJob2xpZGF5cyIsM10=",
    holidayPatch: {
      startDate: "2018-03-12",
      endDate: "2018-03-14"
    }
  }) {
    clientMutationId
  }
}

mutation {
  updateHoliday(input: {
    id: "WyJob2xpZGF5cyIsM10=",
    holidayPatch: {
      startDate: "2018-05-07",
      endDate: "2018-05-07"
    }
  }) {
    clientMutationId
  }
}
````

effacer une période de vacances

````graphql
mutation {
  deleteHoliday(input: {
    id: "WyJob2xpZGF5cyIsMl0="
  }) {
    clientMutationId
  }
}
````

## Test du rechargement à chaud du schéma SQL

ajout de commentaire sur les tables

````sql
comment on table users is 'Les gens de l''équipe des nantastiques';
comment on table holidays is 'les vacances de l''équipe';
````

la doc graphiql est mise à jour automatiquement

ajout d'une nouvelle fonction

````sql
create function check_may_holidays() returns setof holidays as $$
	select distinct id_dgasi
	from public.holidays
	where extract(month from start_date) = 5
	or extract(month from end_date) = 5
$$ language sql stable;
````

appeller la fonction d'interrogation de congés via GraphiQL

````graphql
query {
  checkMayHolidays {
    edges {
      node {
        id
      }
    }
  }
}
````
