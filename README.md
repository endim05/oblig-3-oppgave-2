# oblig-3-oppgave-2

import pandas as pd
import numpy as np
import altair as alt

# begynner med å vaske data
kgdata = pd.read_excel("ssb-barnehager-2015-2023-alder-1-2-aar.xlsx", sheet_name="KOSandel120000",
                       header=3,
                       names=['kom', 'y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23'],
                       na_values=['.', '..'])

for col in ['y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23']:
    mask_over_100 = (kgdata[col] > 100)  # Kan bli et typeproblem?
    kgdata.loc[mask_over_100, col] = float("nan")
kgdata.loc[724:779, 'kom'] = "NaN"
kgdata["kom"] = kgdata['kom'].str.split(" ").apply(lambda x: x[1] if len(x) > 1 else "")
kgdata_meta = kgdata.drop(kgdata.index[724:])

import pandas as pd

def write_pandas_dataframe_to_excel_worksheet_with_nan(pandas_df, file_name, wsh_name):
    with pd.ExcelWriter(file_name, mode='a') as writer:
        pandas_df.to_excel(writer, sheet_name=wsh_name, na_rep='nan')

# printer ut de 10 første linjene med vasket data
print(kgdata_meta.head(10))

def get_top_n(df, column, n=1, ascending=False):
    sorted_df = df.sort_values(by=column, ascending=ascending)
    return sorted_df[['kom', column]].head(n)


import pandas as pd
import numpy as np
import pytest

# Definer get_top_n-funksjonen direkte i testfilen
def get_top_n(df, column, n=1, ascending=False):
    sorted_df = df.sort_values(by=column, ascending=ascending)
    top_n = sorted_df.head(n)
    return top_n[['kom', column]]

# Test for get_top_n funksjonen
def test_get_top_n():
    # Lager en test dataframe
    df = pd.DataFrame({
        'kom': ['A', 'B', 'C', 'D'],
        'y15': [10, 50, 30, 40],
    })

    # Test å hente den øverste raden i synkende rekkefølge
    result = get_top_n(df, 'y15', n=1, ascending=False)
    expected = pd.DataFrame({'kom': ['B'], 'y15': [50]})
    pd.testing.assert_frame_equal(result.reset_index(drop=True), expected)

# kjør test
pytest.main()


import pandas as pd
import numpy as np
import pytest

# Definer get_top_n-funksjonen direkte i testfilen
def get_top_n(df, column, n=1, ascending=False):
    sorted_df = df.sort_values(by=column, ascending=ascending)
    top_n = sorted_df.head(n)
    return top_n[['kom', column]]

# Test for get_top_n funksjonen
def test_get_top_n():
    # Lager en test dataframe
    df = pd.DataFrame({
        'kom': ['A', 'B', 'C', 'D'],
        'y15': [10, 50, 30, 40],
    })

    # Test å hente den øverste raden i synkende rekkefølge
    result = get_top_n(df, 'y15', n=1, ascending=False)
    expected = pd.DataFrame({'kom': ['B'], 'y15': [50]})
    pd.testing.assert_frame_equal(result.reset_index(drop=True), expected)

# kjør test
pytest.main()


A. Highest percentage in 2023:
Salangen: 100.0%

B. Lowest percentage in 2023:
Etne: 55.6%
Highest percentages in 2023:
Salangen: 100.0%
Unjárga-Nesseby: 100.0%
Røst: 100.0%
Lowest percentages in 2023:
Etne: 55.6%
...

# C/E & D. Høyeste og laveste gjennomsnittlige prosent fra 2015 til 2023
# Bruker "mean" funksjonen for år 2015 til 2023, og ignorerer Nan verdier
year_columns = ['y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23']
kgdata_meta['mean_2015_2023'] = kgdata_meta[year_columns].mean(axis=1)

# Avrund til en desimal
kgdata_meta['mean_2015_2023_rounded'] = kgdata_meta['mean_2015_2023'].round(1)

# Funksjon for å få toppverdier med potensielle duplikater
def get_top_values(df, column, top=True, n=3):
    sorted_df = df.sort_values(by=column, ascending=not top)
    top_value = sorted_df[column].iloc[0]
    return sorted_df[sorted_df[column] == top_value][['kom', column]].head(n)

# C/E. Kommune med høyeste gjennomsnittlige prosent fra 2015 til 2023
highest_avg = get_top_values(kgdata_meta, 'mean_2015_2023_rounded')
print("\nC. Kommune(r) med høyeste gjennomsnittlige prosent (2015-2023):")
for index, row in highest_avg.iterrows():
    print(f"{row['kom']}: {row['mean_2015_2023_rounded']:.1f}%")

# D. Kommune med laveste gjennomsnittlige prosent fra 2015 til 2023
lowest_avg = get_top_values(kgdata_meta, 'mean_2015_2023_rounded', top=False)
print("\nD. Kommune(r) med laveste gjennomsnittlige prosent (2015-2023):")
for index, row in lowest_avg.iterrows():
    print(f"{row['kom']}: {row['mean_2015_2023_rounded']:.1f}%")

# G.
'''Henter ut data for den valgte kommunen (municipality) fra DataFrame-en som inneholder data for flere kommuner
og velger data for den valgte kommunen'''
def create_municipality_chart(df, municipality):
    muni_data = df[df['kom'] == municipality]

    # Bruker melt for å smelte datarammen for å få den til riktig format for Altair
    muni_data_melted = pd.melt(muni_data, id_vars=['kom'],
                               value_vars=['y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23'],
                               var_name='year', value_name='percentage')

    # Konverterer year til et riktig datoformat
    muni_data_melted['year'] = muni_data_melted['year'].str.replace('y', '20').astype(int)

    # Lager en chart
    chart = alt.Chart(muni_data_melted).mark_line(point=True).encode(
        x=alt.X('year:O', title='År'),
        y=alt.Y('percentage:Q', title='Prosent', scale=alt.Scale(zero=False)),
        tooltip=['year', 'percentage']
    ).properties(
        title=f'Prosent av barn 1-2 år i barnehage i {municipality} (2015-2023)',
        width=600,
        height=400
    )

    return chart

# G. Opprett og vis et diagram for en valgt kommune
selected_municipality = "Sarpsborg"  # velger å vise et diagram som eksempel
create_municipality_chart(kgdata_meta, selected_municipality)

# Velg en kommune og generer diagrammet for prosent av barn i 1-2 års alderen i barnehage
selected_municipality = "Sarpsborg"  # velger å vise et diagram som eksempel
chart = create_municipality_chart(kgdata_meta, selected_municipality)

# Lagre chart som HTML-fil
chart.save(f"{selected_municipality}_barnehage_prosent_2015_2023.html")
print(f"G. Chart for {selected_municipality} has been saved as an HTML file.")

import pandas as pd
import altair as alt

# Filtrerer kommuner som har data for alle år, og lager en eksplisitt kopi av disse dataene
complete_data = kgdata_meta.dropna(subset=['y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23']).copy()

# Kalkulerer gjennomsnittlige prosent for 2015-2023 ved å bruke .loc
complete_data.loc[:, 'avg_2015_2023'] = complete_data[['y15', 'y16', 'y17', 'y18', 'y19', 'y20', 'y21', 'y22', 'y23']].mean(axis=1)

# Sorterer etter gjennomsnittet og velger topp 10 kommuner
top_10 = complete_data.nlargest(10, 'avg_2015_2023')

# Lager charten
chart = alt.Chart(top_10).mark_bar().encode(
    x=alt.X('kom:N', sort='-y', title='Kommune'),
    y=alt.Y('avg_2015_2023:Q', title='Gjennomsnittlig prosent (2015-2023)'),
    tooltip=['kom', 'avg_2015_2023']
).properties(
    title='Top 10 kommuner med høyest gjennomsnittlig prosent barn i barnehage (1-2 år) 2015-2023',
    width=600,
    height=400
)

# Lagre charten som HTML-fil
chart.save('top_10_kommuner_barnehage_2015_2023.html')
print("H. Chart for top 10 kommuner lagret som 'top_10_kommuner_barnehage_2015_2023.html'")

