# Boletín Triggers

## Todos estos ejercicios se hacen en la tabla de ejemplos SCOTT

## 1. Haz un trigger que solo permita a los vendedores tener comisiones.

```

create or replace trigger controlar_comisiones
before insert or update on emp
for each row
declare
	v_deptno dept.deptno%TYPE;
begin
	select deptno into v_deptno from dept where dname='SALES';

	if :new.job!='SALESMAN' or :new.deptno!=v_deptno and :new.comm is not null then
		raise_application_error(-20100,'Solo los vendedores pueden tener comisiones');
	end if;
end controlar_comisiones;
/

```
